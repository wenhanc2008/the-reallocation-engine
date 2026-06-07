# Mode: opt-timeline-fit -- OPT Timeline-Aware Company Targeting

Use this mode when a student on F-1 OPT visa (expiring within 6–12 months) wants to
identify high-probability H-1B sponsorship targets and understand their timeline
constraints before job searching.

The mode combines sponsorship history (80 Days to Stay layer), job liveness (Job-Ops
layer), and role quality (Cognitive Pivot layer) to rank companies and surface the
specific timeline risk: whether cap-subject H-1B filing window overlaps with the
student's OPT expiration date.

## Required Context

Read first:

- `modes/_shared.md`
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md`
- `data/bls/bls-audit.md`
- `modes/RUN_LOG.md`

## What This Mode Can Verify

This mode can verify:

- which companies have sponsored H-1B visas in the past 3 years (from DOL/USCIS data);
- what approval rate each company achieved (from USCIS H-1B Employer Data Hub);
- whether a job posting is currently live (from HTTP liveness checks);
- what the posted role's skill requirements are (from BLS/O*NET SOC codes);
- whether the student's OPT expiration date overlaps with H-1B cap filing deadlines (calendar math).

This mode **cannot** verify:

- whether this specific company will sponsor this specific student;
- the company's current visa sponsorship policy (may have changed since last filing);
- the actual hiring timeline for this role at this company;
- the student's personal cap-exempt or cap-subject eligibility (requires immigration lawyer);
- salary negotiation outcomes or prevailing wage compliance;
- whether the company will continue sponsoring during H-1B processing delays.

## Primary Tools

**Layer 1: Sponsorship History (80 Days to Stay)**

```bash
# Existing data source — no script required
data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv
```

This file contains 30,369 companies with H-1B sponsorship history (1,557 with records).
Columns used: `company_name`, `state`, `Total Approvals`, `Approval_Rate`, 
`latest_funding_amount`, `latest_funding_date`, `median_salary_offered`, 
`top_job_titles_sponsored`.

**Layer 2: Job Liveness (Job-Ops)**

```bash
# ATS Detection — detect which job posting platform each company uses
python3 SCRIPTS/ats/detect_ats.py \
  --csv data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv \
  --company-column company_name \
  --output data/ats/saloni-targets-ats.csv

# Liveness Check — verify which postings are still live
npm run ats:liveness -- --file data/ats/saloni-job-urls.txt \
  --output data/ats/saloni-liveness.csv
```

**Layer 3: Role Quality (Cognitive Pivot)**

```bash
# Existing data source — no script required
data/bls/compact/soc_occupation_compact.csv
```

This file contains 1,016 occupations with `cognitive_pivot_score` (calculated from
O*NET ability/skill dimensions). Used to assess whether the role has visa narrative
strength (judgment-heavy roles score higher, e.g., 4.0+; routine roles score lower,
e.g., 3.0).

**Timeline Math (Student-Specific)**

No script required. Calculate manually:
- OPT expiration date (given)
- H-1B cap-subject filing window: April 1 – June 30 (fixed annually)
- Overlap: if student graduates after June 30, cap-subject pathway will not work

## Proposed New Data Sources

### 1. H-1B Filing Velocity Score

**What:** Days from job post date to H-1B filing date, averaged per company.

**Why:** Shows how quickly company moves from hiring to visa sponsorship filing.
Fast-moving companies (e.g., Confluent: 14 days average) are safer bets than slow-moving
ones (e.g., Company X: 90 days average) when timeline is tight (6 months remaining).

**Current Status:** NOT IN REPO. Would require:
- Job posting dates from ATS output (e.g., `posted_date` from Greenhouse API)
- H-1B filing dates from DOL LCA records (publicly available, quarterly)
- Joining these two datasets on company name + role SOC code

**Rationale for inclusion:** This metric makes visible which companies move quickly
from hiring to visa filing — crucial for a student with 6-month OPT window. Historical
approval rate says "company sponsors," but velocity says "company sponsors fast enough
for my timeline."

### 2. Company Headcount Momentum

**What:** Percentage change in open roles (target role type, e.g., Product Manager)
over 90-day window.

**Why:** Growing hiring = more likely to sponsor new hires. Shrinking hiring = risky
(company may freeze sponsorships during contraction).

**Current Status:** NOT IN REPO. Would require:
- Historical job posting counts per company per role type
- Tracked weekly or monthly from ATS scan output
- Trend analysis (linear regression or simple % change calculation)

**Rationale for inclusion:** Sponsorship willingness correlates with hiring expansion.
A company filing 10 H-1Bs when hiring 5 PM roles is suspicious (may be strategic
backfilling or sponsoring for specialized roles, not entry PMs). A company filing 10
H-1Bs when hiring 50 PM roles is healthy (1 in 5 get sponsored, sustainable pace).

## Workflow

### Step 1: Establish Timeline Constraints

Input: Student's OPT expiration date and graduation date.

```
OPT expiration: [DATE]
Graduation: [DATE]
Days remaining: [calculated]
Cap-subject filing deadline: April 1 – June 30, [YEAR]
Overlap with graduation? [YES/NO]
```

If graduation is after June 30: **Cap-subject H-1B will not work.** 
Require immigration lawyer verification of cap-exempt eligibility before proceeding.

Output to run log: Timeline risk assessment (see Step 7).

### Step 2: Filter Target Companies

Input: `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`

Filter rows:
- `state` = target geography (e.g., CA, NY, WA)
- `Total Approvals` ≥ 1 (company has sponsored at least once)
- `latest_funding_amount` is not null (company has received VC funding)

Output: Filtered company list for each geography.

Example for Bay Area CA:
```
Total companies in CA: 12,167
Companies with H-1B history: 860
Companies with recent funding: ~600
```

### Step 3: Run ATS Detection

Input: Filtered company list from Step 2.

```bash
python3 SCRIPTS/ats/detect_ats.py \
  --csv data/ats/saloni-targets-ca.csv \
  --company-column company_name \
  --output data/ats/saloni-ca-ats.csv
```

Output: Company list + detected ATS platforms (Greenhouse, Lever, Ashby, or unknown).

Filter to companies with detected platform + at least one open PM-related role.

### Step 4: Check Job Posting Liveness

Input: URLs from Step 3 with detected ATS.

```bash
npm run ats:liveness -- --file data/ats/saloni-ca-job-urls.txt \
  --output data/ats/saloni-ca-liveness.csv
```

Output: URLs marked as Live, Stale (>30 days old), Redirected, or Unknown.

Filter to Live or recently-posted (≤30 days).

### Step 5: Map Roles to SOC Codes and Cognitive Scores

Input: Job titles from Step 4.

Query `data/bls/compact/soc_occupation_compact.csv`:
- Look up job title (e.g., "Product Manager", "Technical Product Manager", "PM")
- Match to SOC code (typically SOC 11-2021 or similar management role)
- Extract `cognitive_pivot_score` (e.g., 3.974)

For "Product Manager" roles:
- SOC 11-2021 (Marketing Managers): cognitive_pivot_score = 3.974 (MODERATE narrative strength)
- Interpretation: Role requires market judgment, strategy, cross-functional coordination
  — work that's harder to offshore. Visa narrative exists but is not as strong as
  specialized technical roles (4.3+).

Output: Company + role + SOC code + cognitive score table.

### Step 6: Score and Rank Companies

Composite score = sponsorship probability (Layer 1) × liveness probability (Layer 2) 
                  × fit probability (Layer 3) × timeline factor

Where:
- **Sponsorship probability** = f(H-1B approvals, approval rate, funding recency)
  - ≥50 approvals + 100% rate = 0.90 (Proven)
  - 10-50 approvals + 90%+ rate = 0.75 (Likely)
  - 1-10 approvals + 80%+ rate = 0.65 (Established)
  - 0 approvals = 0.00 (Avoid)

- **Liveness probability** = f(posting age)
  - <14 days old = 0.95
  - 14-30 days old = 0.70
  - 30+ days old = 0.40
  - Dead/redirect = 0.00

- **Fit probability** = f(student background, role cognitive tier, SOC match)
  - CS background + PM role + cognitive >3.9 = 0.80 (Strong)
  - Non-CS background + PM role + cognitive 3.9 = 0.65 (Moderate)
  - No match = 0.40

- **Timeline factor** = f(hiring timeline vs. OPT deadline)
  - Typical PM hiring 4-6 weeks + 6-month OPT = 1.0 (fits)
  - Typical PM hiring >8 weeks + <3 months OPT = 0.6 (tight)
  - No time = 0.0

**Decision thresholds:**
- Score ≥0.80 = **Apply Now**
- Score 0.65-0.80 = **Apply + Verify** (contact employee first)
- Score 0.40-0.65 = **Lower Priority** (apply only if time permits)
- Score <0.40 = **Skip**

Output: Ranked table (see Output Format below).

### Step 7: Create Timeline Risk Assessment

Input: Student's timeline from Step 1 + scoring from Step 6.

Report:
```
OPT expiration: February 12, 2027 (6 months)
Graduation: August 12, 2026
H-1B cap-subject filing: April 1 – June 30, 2026

TIMELINE RISK ASSESSMENT:
- Graduation is after cap filing closes (August > June)
- Cap-subject pathway will NOT work
- Must verify cap-exempt eligibility BEFORE job search
- If cap-exempt: company can file immediately post-graduation (Aug 2026)
  → Approval by Feb 2027 (within OPT window) ✓
- If NOT cap-exempt: must accept 6-month out-of-status waiting period
  → Not recommended without AOLA or similar arrangement

RECOMMENDATION: Immigration lawyer consultation is mandatory, not optional.
Timeline decision depends entirely on cap-exempt eligibility.
```

### Step 8: Log the Run

Add entry to `modes/RUN_LOG.md` (see Log Template below).

## Output

### Primary Output: Company Ranking Table

Create one table per geography.

```markdown
| Rank | Company | State | H-1B Approvals | Approval Rate | Latest Funding | Posting Age | Liveness | SOC/Cognitive | Composite | Action |
|------|---------|-------|---|---|---|---|---|---|---|---|
| 1 | Confluent Inc | CA | 610 | 100% | 2017-02-21 | 2 days | Live | 11-2021 / 3.97 | 0.88 | Apply Now |
| 2 | Datadog Inc | NY | 340 | 100% | 2015-12-28 | 3 days | Live | 11-2021 / 3.97 | 0.86 | Apply Now |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
```

Rank companies by composite score within each geography.

### Secondary Output: Timeline Risk Assessment

See Step 7 above. Include as text document (not table) with calendar-based analysis.

### Explicit Verification Status

For each company-role pair, report:

**VERIFIED** (from government data or HTTP response):
- H-1B approval count (from USCIS)
- Approval rate (from USCIS)
- Funding amount and date (from SEC EDGAR)
- Job posting liveness (from HTTP 200/404 response)
- Role SOC code and cognitive score (from BLS/O*NET)
- Student's OPT expiration date (from I-20)

**INFERRED** (requires human follow-up):
- Whether company will sponsor this student (mitigation: contact employee)
- Actual hiring timeline for this role at this company (mitigation: Glassdoor + network)
- Student's cap-exempt eligibility (mitigation: immigration lawyer)
- Whether salary offer matches prevailing wage (mitigation: BLS prevailing wage lookup)
- Whether company will file H-1B immediately after offer (mitigation: offer negotiation)

## Log Template

Add to `modes/RUN_LOG.md`:

```markdown
## YYYY-MM-DD — OPT timeline evaluation: [Geography]

- **Mode:** opt-timeline-fit
- **Student:** [Name] | OPT expiration: [DATE]
- **Target role:** [Role] (SOC: [CODE])
- **Geography:** [State(s)]
- **Input files:**
  - `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`
  - `data/bls/compact/soc_occupation_compact.csv`
  - Company list: [file]
- **Scripts run:**
  - `SCRIPTS/ats/detect_ats.py` → [output file]
  - `npm run ats:liveness` → [output file]
- **Outputs:**
  - Company ranking table: [file]
  - Timeline risk assessment: [file]
  - Application target list: [N companies, N "Apply Now", N "Apply + Verify"]
- **Worked:** [What succeeded, e.g., "ATS detection found Greenhouse at 45 companies; liveness checks fast; SOC mapping straightforward"]
- **Did not work:** [What failed or is missing, e.g., "H-1B filing velocity data not in repo; requires manual lookup"]
- **Blockers:** [What prevents next step, e.g., "Student has not verified cap-exempt eligibility; proceeding without this is risky"]
- **Next:** [What to do next, e.g., "Run same workflow for Seattle WA, Bay Area CA, NYC metro; then consolidate ranking across all geographies"]
```

## Prompting Rule

Only use LLM judgment **after** script output has been inspected and verified data has been
checked.

The LLM may:
- Summarize which companies passed each filter
- Explain likely reasons for liveness failures
- Draft timeline risk language based on calendar math
- Propose next steps for company research

The LLM must **not**:
- Invent H-1B approval counts or rates
- Guess at hiring timelines
- Assess cap-exempt eligibility (this requires a lawyer)
- Claim a company "will probably sponsor" without checking DOL records first
- Recommend a company without checking both sponsorship history AND liveness

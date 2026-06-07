# Mode Assignment Submission
## fullstack-swe-sponsor-triage
**Student:** REDACTED
**Program:** MS Information Systems, Northeastern University Seattle
**Date:** 2026-06-03

---
---

# PART 1 — MODE FILE
# `modes/fullstack-swe-sponsor-triage.md`

---

# Mode: fullstack-swe-sponsor-triage — Pre-OPT Full-Stack/Backend SWE Sponsor Triage

**Status:** New mode — partially uses existing scripts; one proposed script marked clearly below.
**When to use:** Before OPT begins, when you have a list of target companies and need to separate
proven H-1B sponsors from companies that merely claim they will sponsor — so you invest interview
effort only where sponsorship history, active hiring, role fit, and location are all confirmed.

---

## Who This Mode Is For

An international master's student in Information Systems on F-1 visa, graduating August 2026,
with OPT starting October 2026. Target roles are full-stack or backend Software Engineer positions
(SOC 15-1252) at mid-size to large tech companies in Seattle WA, San Francisco Bay Area CA,
or Los Angeles CA. The student has prior SWE experience and is not applying as a fresh graduate — the
bottleneck is not interview readiness, it is identifying which companies are worth pursuing given
the H-1B cap lottery deadline of April 2027 (FY2028 cap).

**Do not use this mode for:**
- Companies already in active interview pipeline (use `modes/oferta.md` per role instead)
- Cap-exempt employers such as universities or nonprofits
- Roles outside SOC 15-1252 — sponsorship history is SOC-specific

---

## Timeline Context

| Milestone | Date |
|---|---|
| Graduation | August 2026 |
| OPT start | October 2026 |
| Standard OPT end | October 2027 |
| STEM OPT extension end | October 2029 |
| H-1B cap lottery (FY2028) | April 2027 |
| Latest date offer must arrive for standard processing | ~December 2026 |

---

## Required Context — Read First

Before running anything:

- `modes/_shared.md` — verified-data contract and logging rules
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md` — understand what sponsorship
  columns are reliable before querying
- `data/bls/bls-audit.md` — understand BLS/SOC coverage before using role-quality scores
- `data/ats/pipeline.md` if it exists — check what is already in the pipeline
- `modes/RUN_LOG.md` — check if this company list has been checked before

---

## What This Mode Can Verify

| Claim | Source | Status |
|---|---|---|
| Company has filed H-1B petitions for SOC 15-1252 | `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv` | ✅ Verified |
| H-1B approval rate and median salary offered | `SEC_DOL_H1b_data_mapped.csv` columns: `Approval_Rate`, `median_salary_offered` | ✅ Verified |
| Top job titles sponsored by company | `SEC_DOL_H1b_data_mapped.csv` column: `top_job_titles_sponsored` | ✅ Verified |
| ATS provider used by company | `SCRIPTS/ats/detect_ats.py` | ✅ Verified |
| Number of open jobs at time of check | `SCRIPTS/ats/detect_ats.py` `open_job_count` field | ✅ Verified |
| Job posting URL is live at time of check | `npm run ats:liveness` | ✅ Verified at time of check only |
| SOC 15-1252 cognitive demand and labor-market score | `data/bls/compact/soc_occupation_compact.csv` | ✅ Verified |

## What This Mode Cannot Verify

| Claim | Why |
|---|---|
| Company will sponsor H-1B in 2027 specifically | Past petitions do not guarantee future behavior — freezes, layoffs, policy changes are not in the dataset |
| A specific open role will be included in H-1B petition | LCA/H-1B data is at employer level, not role level |
| USCIS will process petition before April 2027 lottery | Processing times are estimates; RFEs or backlogs are not predictable |
| Posting is still accepting applications | Liveness check confirms URL resolves — it cannot confirm the ATS queue is open |
| Company office is in Seattle, SF Bay Area, or LA | Location data in dataset is city/state only — office-level filtering requires manual check |

---

## Workflow

### Step 1 — Build Your Company List

```bash
touch data/ats/fullstack-swe-targets.txt
```

Add one company per line. Keep to 10–30 companies for a manageable first run.

---

### Step 2 — Detect ATS Provider

```bash
cd SCRIPTS/ats
python3 detect_ats.py \
  --file ../../data/ats/fullstack-swe-targets.txt \
  --output ../../data/ats/fullstack-swe-ats-check.csv
cd ../..
```

Or run against the full SEC/H-1B mapped dataset:

```bash
cd SCRIPTS/ats
python3 detect_ats.py \
  --csv ../../data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv \
  --company-column company_name \
  --output ../../data/ats/ats_detection.csv
cd ../..
```

Note: Large companies (Amazon, Google, Apple, Microsoft) use internal ATS systems.
`not_found` means the detector could not confirm — not that there are no jobs.

---

### Step 3 — Check Sponsorship History

Run the SEC data audit first:

```bash
python3 SCRIPTS/audit_sec_dol_h1b_data.py
```

Then query `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`. Focus on:

| Column | What It Tells You |
|---|---|
| `Total Approvals` | How many H-1B petitions approved historically |
| `Approval_Rate` | Approval rate — low rate may indicate RFE risk |
| `median_salary_offered` | Median LCA salary — use as floor check |
| `top_job_titles_sponsored` | Whether SWE titles appear — not just compliance or legal |
| `latest_funding_amount` | Most recent SEC Form D amount |
| `latest_funding_date` | Recency of funding |

Report each company as:
- `Proven` — strong H-1B history with SWE titles confirmed
- `Likely` — approvals present but count is low or titles unclear
- `Flagged` — strong history but known external policy change — verify manually
- `Unknown` — not found in dataset

---

### Step 4 — Check Job Posting Liveness

```bash
npm run ats:liveness -- --file data/ats/fullstack-swe-urls.txt
```

Update `data/ats/pipeline.md`:

```markdown
## Pending
- [ ] https://boards.greenhouse.io/stripe/jobs/456 | Stripe | Backend Engineer

## Processed
- [x] 2026-06-04 | https://boards.greenhouse.io/stripe/jobs/456 | Stripe | Backend Engineer | live | sponsor: Proven

## Problems
- [!] 2026-06-04 | https://boards.greenhouse.io/example/789 | Example Co | SWE | redirected to careers home
```

```bash
npm run ats:verify
```

---

### Step 5 — Pull BLS/SOC Role Quality Score

```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py
```

Read `data/bls/compact/soc_occupation_compact.csv` and filter for SOC 15-1252.

---

### Step 6 — H-1B Lottery Window Check (Proposed — Not Yet Implemented)

**This script does not exist yet. Output below is manually calculated.**

Proposed script: `SCRIPTS/sec/opt_lottery_window.py`

```bash
# PROPOSED — does not exist yet
python3 SCRIPTS/sec/opt_lottery_window.py \
  --opt-start 2026-10-01 \
  --ead-expiry 2027-10-01 \
  --lottery-date 2027-04-01 \
  --input data/ats/fullstack-swe-sponsor-output.csv \
  --output data/ats/fullstack-swe-tiered.csv
```

Until this script exists, calculate manually:
- April 2027 lottery → petition must be filed by ~March 31, 2027
- Standard H-1B processing: ~3–4 months → offer needed by ~December 2026
- Premium processing (15 business days): offer needed by ~March 2027

---

### Step 7 — Evaluate Top Roles Individually

For each company that passes all steps, run a full role evaluation:

```text
data/ats/reports/YYYY-MM-DD-company-role-evaluation.md
```

---

## Output Format

Final output file: `data/ats/fullstack-swe-sponsor-output.csv`

| Column | Source | Verified? |
|---|---|---|
| `company_name` | Student input | — |
| `h1b_total_approvals` | `SEC_DOL_H1b_data_mapped.csv` | ✅ Verified |
| `h1b_approval_rate` | `SEC_DOL_H1b_data_mapped.csv` | ✅ Verified |
| `top_job_titles_sponsored` | `SEC_DOL_H1b_data_mapped.csv` | ✅ Verified |
| `median_salary_offered` | `SEC_DOL_H1b_data_mapped.csv` | ✅ Verified |
| `latest_funding_amount` | `SEC_DOL_H1b_data_mapped.csv` | ✅ Verified |
| `ats_provider` | `SCRIPTS/ats/detect_ats.py` | ✅ Verified |
| `open_job_count` | `SCRIPTS/ats/detect_ats.py` | ✅ Verified |
| `posting_live` | `npm run ats:liveness` | ✅ Verified at time of check |
| `soc_code` | Student-assigned from role title | ⚠️ Inferred |
| `bls_cognitive_score` | `data/bls/compact/soc_occupation_compact.csv` | ✅ Verified |
| `sponsorship_tier` | Derived from above columns | ⚠️ Inferred |
| `lottery_priority_tier` | `opt_lottery_window.py` (proposed) | 🔲 Not yet implemented |

---

## Log Template for `modes/RUN_LOG.md`

```markdown
## YYYY-MM-DD — fullstack-swe-sponsor-triage

- **Mode:** fullstack-swe-sponsor-triage
- **Student profile:** MS Information Systems, F-1 OPT start Oct 2026,
  targeting full-stack/backend SWE (SOC 15-1252), Seattle / SF Bay Area / Los Angeles
- **Inputs:** data/ats/fullstack-swe-targets.txt (N companies)
- **Commands run:**
  - `python3 SCRIPTS/ats/detect_ats.py --file ...`
  - `python3 SCRIPTS/audit_sec_dol_h1b_data.py`
  - `python3 query_companies.py`
  - `npm run ats:liveness -- --file ...`
  - `npm run ats:verify`
  - `python3 SCRIPTS/bls/extract_soc_occupation_table.py`
- **Outputs:**
  - `data/ats/fullstack-swe-ats-check.csv`
  - `data/ats/fullstack-swe-sponsor-output.csv`
  - `data/ats/pipeline.md` (updated)
- **Sponsorship results:** N Proven / N Likely / N Flagged / N Unknown
- **ATS results:** N Greenhouse / N Lever / N not_found (custom ATS)
- **Liveness results:** N live / N stale / N redirected
- **Proposed scripts not yet run:** opt_lottery_window.py
- **Open issues:** [anything that blocked or surprised you]
- **Next:** Run oferta.md on TIER_1 companies
```

---
---

# PART 2 — DOMAIN JUSTIFICATION

---

## Who Uses This Mode and When

This mode is designed for an international master's student in Information Systems on F-1 visa,
graduating August 2026, with OPT authorization beginning October 2026. The student has 3+ years
of prior software engineering experience in Java, Python, TypeScript, and cloud infrastructure,
and is targeting full-stack or backend SWE roles (SOC 15-1252) at mid-size to large tech companies
in Seattle WA, San Francisco Bay Area CA, or Los Angeles CA.

The mode is used in the window between graduation and OPT start — roughly August to October 2026
— when the student is building a prioritized company list before active applications begin. The
H-1B FY2028 cap lottery opens in April 2027, meaning a petition must be filed before that date.
Working backward, an offer must arrive no later than approximately December 2026 for standard H-1B
processing. This gives the student a real but bounded window — wasting two months in a pipeline at
a company that has never filed an H-1B petition is a material cost.

---

## Information Asymmetry Addressed

**Layer 1 — Sponsorship claims are unverified by default.**
Nearly every mid-to-large tech company states "we sponsor work authorization" on their careers
page. This is not verifiable without checking DOL Labor Condition Application records. A company
that sponsored 40 H-1B petitions for software engineers is categorically different from one that
sponsored 2 petitions for compliance staff. The student cannot see this difference from a job board.
The 80 Days dataset makes this visible — but only if the student knows to check it and knows which
columns to trust. This mode makes that check the first filter.

**Layer 2 — Funding recency signals hiring stability.**
A company that raised a Series D eighteen months ago is more likely to have stable engineering
headcount than one whose last Form D filing was three years ago. The `latest_funding_amount` and
`latest_funding_date` columns in the mapped dataset surface this without requiring the student to
manually search EDGAR for each company.

**Layer 3 — Job posting liveness is not guaranteed by a URL existing.**
Students routinely spend time tailoring applications to postings that are no longer accepting
submissions. The liveness check via `npm run ats:liveness` is a weak but verified signal that
a real hiring pipeline exists for the role being targeted.

---

## Connection to Engine Layers

**80 Days to Stay** is the primary layer. H-1B petition history and SEC Form D funding data
provide the foundational company triage. Without this layer, the student is sorting companies
by brand recognition — a proxy with no verified connection to sponsorship behavior.

**Job-Ops** is the secondary layer. ATS detection via `SCRIPTS/ats/detect_ats.py` and liveness
checking via `npm run ats:liveness` ensure the student is spending time on postings that are
actually in motion. The ATS detector also returns `open_job_count` — a real-time signal of
how actively a company is hiring at the time of the check.

**The Cognitive Pivot** is the tertiary layer. SOC 15-1252 (Software Developers) scores
consistently high on cognitive demand in the BLS/O*NET compact table — full-stack and backend
SWE roles are among the more resilient occupations to AI substitution. For a student who will
need H-1B renewal in 3 years and potentially green card sponsorship beyond that, targeting a
role category with strong labor-market staying power matters.

---

## Failure Modes

**Failure Mode 1 — LCA data lags a policy change by 12 to 18 months.**
This is the hardest error to catch. A company with strong historical H-1B approvals looks like
a top-tier sponsor in the dataset. But if that company has recently frozen new H-1B sponsorship
due to a layoff or external policy shift, the dataset shows nothing. The student, seeing the
strong historical signal, allocates significant interview effort and learns the truth only in a
late-stage recruiter call — or never. This error is hardest to catch for a student without a
professional network inside the company who could surface the change informally.

**Failure Mode 2 — H-1B history at employer level does not confirm sponsorship for the
specific role level the student is targeting.**
The LCA data is aggregated at the employer level. A large company may show hundreds of H-1B
approvals — but the `top_job_titles_sponsored` column may reveal those are predominantly
Senior and Staff Engineer titles. A student targeting new-grad or early-career roles may find
that the company's actual sponsorship practice for that level is far thinner. This error is
hardest to catch for a student who sees a high total approval count and treats it as blanket
confirmation without reading the job title breakdown carefully.

---
---

# PART 3 — WORKED EXAMPLE

---

**Date run:** 2026-06-04
**Student profile:** MS Information Systems, Northeastern University Seattle,
graduating August 2026. OPT start: October 2026. EAD expiry: October 2027.
STEM OPT extension eligible: October 2027–October 2029.
Target roles: Full-Stack / Backend Software Engineer (SOC 15-1252).
H-1B cap lottery target: FY2028 (April 2027).
Latest offer deadline for standard processing: ~December 2026.

---

## Inputs

**Company list:** `data/ats/fullstack-swe-targets.txt` — 15 companies

```
Microsoft, Amazon, Airbnb, Chewy, Databricks, Stripe, Redfin,
Zillow, Salesforce, Figma, Google, Meta, LinkedIn, Walmart, Apple
```

---

## Step 1 — ATS Detection (actually run)

**Command:**
```bash
cd SCRIPTS/ats
python3 detect_ats.py "Microsoft" "Stripe" "Airbnb" "Databricks" "Figma" "LinkedIn"
python3 detect_ats.py "Amazon" "Chewy" "Redfin" "Zillow" "Salesforce" "Google" "Meta" "Walmart" "Apple"
```

**Real output from script (2026-06-04):**

| Company | ATS Detected | Open Jobs | Detection Status |
|---|---|---|---|
| Stripe | Greenhouse | 485 | ✅ found |
| Airbnb | Greenhouse | 233 | ✅ found |
| Databricks | Greenhouse | 759 | ✅ found |
| Figma | Greenhouse | 162 | ✅ found |
| LinkedIn | Greenhouse | 53 | ✅ found |
| Microsoft | — | 0 | not_found (custom ATS) |
| Amazon | — | 0 | not_found (custom ATS) |
| Chewy | — | 0 | not_found (custom ATS) |
| Redfin | — | 0 | not_found (custom ATS) |
| Zillow | — | 0 | not_found (custom ATS) |
| Salesforce | — | 0 | not_found (custom ATS) |
| Google | — | 0 | not_found (custom ATS) |
| Meta | — | 0 | not_found (custom ATS) |
| Walmart | — | 0 | not_found (custom ATS) |
| Apple | — | 0 | not_found (custom ATS) |

`not_found` means the detector could not confirm Greenhouse/Lever — not that there are no jobs.
These companies use internal ATS systems outside the detector's scope.

---

## Step 2 — Sponsorship History Check (actually run)

**Command:**
```bash
python3 query_companies.py
# queries data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv
```

**Real output from CSV (2026-06-04):**

| Company | Total Approvals | Approval Rate | Median Salary | Top Titles Sponsored | In Dataset |
|---|---|---|---|---|---|
| Airbnb | 1,000 | 99.0% | $158,080 | Software Engineer, Senior SWE, Data Scientist | ✅ Yes |
| Databricks | 1,640 | 99.5% | $149,422 | Software Engineer, Senior SWE, Solutions Architect | ✅ Yes |
| Figma | 188 | 98.9% | $190,000 | Software Engineer, Software Engineer - Infrastructure | ✅ Yes |
| LinkedIn | 4,962 | 99.4% | $167,149 | Sr Software Engineer, Staff SWE, Software Engineer | ✅ Yes |
| Microsoft | 12,226 | 96.6% | $156,705 | Software Engineer, Product Manager, TPM | ✅ Yes |
| Stripe | 1,250 | 98.3% | $158,371 | Software Engineer, Backend Engineer | ✅ Yes |
| Salesforce | 66 | 86.8% | $143,998 | Technical Program Manager | ✅ Yes (⚠️ TPM only) |
| Amazon | — | — | — | — | ❌ Not in dataset |
| Chewy | — | — | — | — | ❌ Not in dataset |
| Redfin | — | — | — | — | ❌ Not in dataset |
| Zillow | — | — | — | — | ❌ Not in dataset |
| Google | — | — | — | — | ❌ Not in dataset |
| Meta | — | — | — | — | ❌ Not in dataset |
| Walmart | — | — | — | — | ❌ Not in dataset |
| Apple | — | — | — | — | ❌ Not in dataset |

**7 of 15 companies found in the repo dataset. 8 not found — documented as an honest gap.**

---

## Step 3 — BLS Role Quality Check (verified)

```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py
```

| SOC Code | Occupation | Cognitive Demand | 10-yr Growth | Median Wage |
|---|---|---|---|---|
| 15-1252 | Software Developers | High | +26% | $132,270 |

All 15 target roles fall in SOC 15-1252. No role deprioritized on labor-market grounds.

---

## Step 4 — Lottery Window Math (simulated — proposed script not yet implemented)

`SCRIPTS/sec/opt_lottery_window.py` does not exist yet. Calculated manually:

| Deadline | Date |
|---|---|
| FY2028 H-1B cap lottery | April 1, 2027 |
| Latest offer — standard processing | ~December 2026 |
| Latest offer — premium processing | ~March 2027 |

All companies with verified sponsorship history need to be in active pipeline
from October 2026. There is no TIER_2 "pursue later" queue.

---

## Final Output Table

| Company | H-1B Approvals | Rate | Median Salary | ATS | SWE Titles | Sponsorship | Action |
|---|---|---|---|---|---|---|---|
| Databricks | 1,640 | 99.5% | $149,422 | Greenhouse 759 jobs | ✅ Yes | Proven | Pursue |
| LinkedIn | 4,962 | 99.4% | $167,149 | Greenhouse 53 jobs | ✅ Yes | Proven | Pursue |
| Airbnb | 1,000 | 99.0% | $158,080 | Greenhouse 233 jobs | ✅ Yes | Proven | Pursue |
| Stripe | 1,250 | 98.3% | $158,371 | Greenhouse 485 jobs | ✅ Yes | Proven | Pursue |
| Figma | 188 | 98.9% | $190,000 | Greenhouse 162 jobs | ✅ Yes | Proven | Pursue |
| Microsoft | 12,226 | 96.6% | $156,705 | not_found (custom) | ✅ Yes | Proven | Pursue — manual ATS check |
| Salesforce | 66 | 86.8% | $143,998 | not_found (custom) | ⚠️ TPM only | Weak | Investigate — no SWE titles found |
| Amazon/Google/Meta/Apple/Walmart/Chewy/Redfin/Zillow | — | — | — | not_found | Unknown | Not in dataset | Manual DOL lookup needed |

---

## Verified vs. Inferred

| Claim | Status | Source |
|---|---|---|
| ATS provider and open job count | ✅ Verified | `SCRIPTS/ats/detect_ats.py` — actually run 2026-06-04 |
| H-1B approval counts, rates, salaries, job titles | ✅ Verified | `SEC_DOL_H1b_data_mapped.csv` — actually queried 2026-06-04 |
| BLS cognitive demand score | ✅ Verified | `data/bls/compact/soc_occupation_compact.csv` |
| SOC code 15-1252 assigned to all roles | ⚠️ Inferred | From role title — not verified against actual petition |
| Lottery priority tiers | 🔲 Simulated | `opt_lottery_window.py` not yet implemented — calculated manually |
| Company will sponsor in FY2028 | ❌ Not verified | Past petitions only — no guarantee of future behavior |
| 8 companies not in dataset | ✅ Verified | Script returned no matching rows — honest gap |

---

## What I Would Change After Seeing the Output

**1. Salesforce is the most important finding.**
Salesforce looks like a major tech employer. The data says only 66 H-1B approvals at 86.8%
approval rate, and the top sponsored title is Technical Program Manager — not Software Engineer.
Zero Software Engineer titles appear in the dataset. From the outside this company looks like
a strong target. Verified data says it is a weak SWE sponsor. This is exactly the asymmetry
the mode is designed to surface — and it would have been invisible from the careers page alone.

**2. 8 of 15 companies not in the repo dataset is a real gap.**
Amazon, Google, Meta, Apple, Walmart, Chewy, Redfin, and Zillow all returned no matching rows.
This does not mean they don't sponsor — it means the 80 Days dataset does not cover them at the
resolution needed. The correct next step is a manual lookup on the DOL public LCA disclosure
portal for each company before treating them as unknown. This should be documented as a
proposed addition to the mode: a `manual_lca_lookup` step for companies not found in the dataset.

**3. The proposed `opt_lottery_window.py` script would reduce errors.**
The timeline math is straightforward but error-prone when done manually. A simple CLI tool
taking `--opt-start` and `--ead-expiry` as inputs would make this mode more reliable across
students with different OPT timelines.

**4. The `not_found` ATS result for 10 companies needs a manual fallback.**
The detector only checks Greenhouse and Lever. 10 of 15 companies use internal ATS systems.
A proposed addition: a lookup table of known internal ATS URLs for large companies
(jobs.amazon.com, careers.google.com, etc.) so the student knows exactly where to verify
rather than treating `not_found` as an unknown.


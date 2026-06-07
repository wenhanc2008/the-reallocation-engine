# Mode: new-grad-platform-data-eng — New-Grad Platform Data Engineering Sponsor Filter

Use this mode when targeting **Platform/Infrastructure Data Engineer** roles as a new graduate (0-1 years experience) who needs H-1B sponsorship and wants to focus on companies that actually hire at the entry level, not just senior engineers.

## Required Context

Read first:

- `modes/_shared.md`
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`
- `data/bls/compact/soc_occupation_compact.csv`
- `modes/RUN_LOG.md`

## What This Mode Can Verify

This mode can verify:

- **H-1B approval volume and rates** from DOL LCA and USCIS records — companies with >500 approvals signal regular, consistent hiring rather than occasional senior-only sponsorship
- **Funding recency and amount** from SEC Form D filings — recent funding (last 18 months) suggests active growth and hiring capacity
- **Median salaries** offered by companies for their sponsored roles (from H-1B LCA disclosure data)
- **SOC 15-1243** (Database Architects / Data Engineers) occupation statistics: median salary ($135,980), employment trends, and cognitive demand scores
- **Cognitive pivot score** (3.933 for SOC 15-1243) indicating reasonable resilience to AI automation

## What This Mode Cannot Verify

This mode cannot verify:

- Whether specific job postings are for **entry-level vs. senior** roles — H-1B approval data shows job titles like "Software Engineer" but doesn't distinguish experience requirements
- Whether companies are **currently hiring** — the data shows historical sponsorship patterns, not real-time hiring status
- **Company culture** or how welcoming they are to new graduates — approval counts suggest hiring volume but not environment
- **Exact H-1B processing timelines** — we see approval rates but not how fast individual companies process petitions
- Whether sponsored roles are specifically for **Platform/Infrastructure** work vs. other engineering disciplines

## Primary Tools

Use existing repo data and commands:

```bash
# Find specific data infrastructure companies in the mapped dataset
grep -iE "databricks|snowflake|confluent|datadog" \
  data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv

# Check occupation data for Data Engineer SOC code
grep "15-1243" data/bls/compact/soc_occupation_compact.csv

# Detect ATS platform for target companies (if known)
cd SCRIPTS/ats
python3 detect_ats.py "Databricks, Inc." "Snowflake Inc."

# Scan for live job postings (requires portals.yml configuration)
npm run ats:scan

# Check if job URLs are currently live or stale
npm run ats:liveness -- --file data/ats/job-urls.txt

# Verify pipeline data integrity
npm run ats:verify
```

## Workflow

### Step 1: Filter for High-Volume Data Infrastructure Sponsors

Open or grep `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`:

```bash
# Example: Find companies with high approval volume
grep -iE "databricks|snowflake|confluent" \
  data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv
```

**Filter criteria:**
- `Total Approvals` ≥ 500 (signals consistent, regular hiring)
- `Approval_Rate` ≥ 95% (low visa denial risk)
- Focus on data-infrastructure companies: Databricks, Snowflake, Confluent, Datadog, Fivetran, dbt Labs, etc.

**Rationale:** A company that has sponsored 500+ H-1Bs over several years is hiring at scale. This increases the probability they have entry-level positions, not just senior-only roles. Small startups with 10-20 approvals may only sponsor experienced engineers.

### Step 2: Check Funding Recency

For each company identified in Step 1, check:
- `latest_funding_date` — prioritize companies with funding in last 18 months
- `latest_funding_amount` — larger amounts (>$50M) suggest sustained hiring capacity

**Rationale:** Recent funding signals active growth mode. A company that raised $100M two years ago may have already completed their hiring wave. Recent funding suggests they're hiring NOW.

**Note:** Post-IPO companies (like Snowflake) won't show recent funding but may still hire heavily. Cross-check against approval volume — if they have >1000 approvals despite old funding dates, they're still active.

### Step 3: Verify Role Quality with BLS Data

```bash
grep "15-1243" data/bls/compact/soc_occupation_compact.csv
```

**Check:**
- `annual_median_wage` — expect $120k-$180k for data infrastructure roles
- `cognitive_pivot_score` — score >3.5 indicates reasonable AI resistance
- `employment` — total employment in this occupation nationwide

**Rationale:** Confirms that Platform Data Engineering is a viable long-term career path with competitive salaries and demand for human judgment/reasoning.

### Step 4: Scan for Current Openings (Optional but Recommended)

If time permits and you've configured `data/ats/portals.yml`:

```bash
npm run ats:scan
npm run ats:liveness
```

**What this adds:** Verifies that companies with strong historical sponsorship are actually posting roles NOW, not just in the past. Helps catch companies that have frozen hiring despite good historical data.

### Step 5: Prioritize and Rank

Create a ranking based on:

1. **Primary:** H-1B approval volume (higher = more likely to have entry-level roles)
2. **Secondary:** Approval rate (>95% preferred)
3. **Tertiary:** Funding recency (<18 months) OR very high approval volume (>1000)

**Priority tiers:**
- **HIGH:** >1000 approvals + >95% rate + (recent funding OR post-IPO growth signal)
- **MEDIUM:** 500-1000 approvals + >95% rate
- **LOW:** <500 approvals (may only hire senior, skip unless you have a referral)

### Step 6: Log Results

Update `modes/RUN_LOG.md` with what you found, what you're prioritizing, and why.

## Output Format

Return a table like this:

| Company | H-1B Approvals | Approval Rate | Latest Funding | Median Salary | Priority |
|---------|----------------|---------------|----------------|---------------|----------|
| Databricks | 1640 | 99.5% | Sept 2025 ($1.07B) | $149,423 | HIGH |
| Snowflake | 1816 | 98.5% | Feb 2020 (now public) | $170,000 | HIGH |
| Confluent | 610 | 100% | Feb 2017 (now public) | $165,000 | MEDIUM |
| Datadog | 340 | 100% | Dec 2015 (now public) | $181,000 | MEDIUM |

**Priority explanation:**
- **HIGH** = Definitely apply — volume + rate indicate they hire regularly, likely have entry-level openings
- **MEDIUM** = Research first — check if job descriptions mention "new grad" or "entry-level" before applying
- **LOW** = Skip unless you have a referral — low volume suggests senior-only hiring

## Decision Rule

- **HIGH priority companies:** Apply immediately. High approval volume (>1000) is a strong signal they hire at multiple levels.
- **MEDIUM priority companies:** Apply only if job descriptions explicitly mention "new grad," "entry-level," or "0-2 years experience."
- **LOW priority companies (<500 approvals):** Skip unless you have a direct referral or insider knowledge they hire new grads.

## What the Machine Could Not Know

This mode filters on approval volume as a proxy for "new-grad friendly." It cannot know:

- A company with 1500 approvals might only sponsor PhDs or 5+ years experience
- A company with 400 approvals might have a dedicated new-grad program
- Post-IPO companies (old funding dates) may still be hiring aggressively
- A company froze hiring last month despite years of strong sponsorship history

**Human override:** If you have insider information (referral, recruiter contact, explicit new-grad posting) that contradicts the mode's priority, trust your specific information over the statistical signal.

## Log Template

```markdown
## YYYY-MM-DD — New-Grad Platform Data Eng Filter

- **Mode:** new-grad-platform-data-eng
- **Input:** data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv
- **Filter criteria:** >500 H-1B approvals, >95% approval rate, data infrastructure focus
- **Companies identified:** [Databricks, Snowflake, Confluent, etc.]
- **Top 3 HIGH priorities:** [list with reasoning]
- **Next action:** [Apply to X, scan Y careers page for live postings, etc.]
- **Skipped:** [List of companies with <500 approvals and why]
```

## Prompting Rule

Only use LLM judgment AFTER data has been checked. The LLM may:
- Help draft application materials based on verified company information
- Explain technical concepts from job descriptions
- Summarize company tech stacks from public sources

The LLM must NOT:
- Invent H-1B approval statistics
- Guess at funding amounts or dates
- Claim to know if a company is "new-grad friendly" without citing data
- Make up median salary figures

Every recommendation must trace back to a number in the SEC_DOL_H1b_data_mapped.csv file, the BLS occupation data, or a live job posting URL.

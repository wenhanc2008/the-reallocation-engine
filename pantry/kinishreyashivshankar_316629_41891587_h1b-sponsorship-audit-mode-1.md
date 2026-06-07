Mode: h1b-sponsorship-audit -- H-1B Sponsorship Audit
Current status: draft workflow.

Use this after accepting an offer at an IT consulting firm on OPT. This
mode does not help you find a job. It audits whether your current employer
is a reliable H-1B sponsor and keeps a warm backup list before your OPT
expires.

## Required Context

* `modes/_shared.md`
* `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`
* `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md`
* `data/bls/compact/soc_occupation_compact.csv`
* `data/bls/bls-audit.md`
* `data/sec/form-d/processed/` — most recent quarter
* `modes/RUN_LOG.md`

## When to Run

* Within 90 days of starting at a consulting or staffing firm on OPT
* At the 12-month mark before H-1B lottery season (January–March)
* Any time your employer announces layoffs, a contract loss, or
  restructuring

## Verified Sequence

**Step 1 — Audit the mapped dataset**

```bash
python3 SCRIPTS/audit_sec_dol_h1b_data.py
```

Read the generated `-audit.md` before querying. Confirms row count,
H-1B coverage rate, and column names. Do not skip this step.

Then search for your employer:

```bash
grep -i "EMPLOYER NAME" \
  data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv
```

Key columns to record if found:
`Total Approvals`, `Total Denials`, `Approval_Rate`,
`median_salary_offered`, `top_job_titles_sponsored`,
`latest_funding_amount`, `latest_funding_date`

If not found, note it as a data gap — not a red flag. The audit
confirms H-1B data is present for only ~5.1% of the 30,369 companies.
Large consulting firms that are not VC-backed are frequently absent.
Try name variants (LLC, Inc, parent entity) before concluding no
record exists.

**Step 2 — BLS role resilience check**

Read the audit first:

```bash
data/bls/bls-audit.md
```

Then extract your target SOC codes:

```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py
grep -E "SOC-CODE" data/bls/compact/soc_occupation_compact.csv
```

Relevant SOC codes for SAP and software consultants:

| SOC Code   | Title                                   |
|------------|-----------------------------------------|
| 15-1252.00 | Software Developers                     |
| 15-1299.08 | Computer Systems Engineers/Architects   |
| 15-1211.00 | Computer Systems Analysts               |

Record `cognitive_pivot_score`. The compact table covers 1,016
occupations. Scores above 6.5 indicate AI-resilient work requiring
judgment, verification, and systems reasoning. Prioritize backup roles
with higher scores — architecture and integration design over
configuration and ticket resolution.

**Step 3 — Backup employer ATS scan**

```bash
cd SCRIPTS/ats
python3 detect_ats.py "Company A" "Company B" "Company C"
```

Note: current scrapers support Greenhouse and Lever only. Ashby and
Workable are not yet implemented. If a company is not found on either
platform, record as not detected — not as confirmed no ATS.

Then run liveness checks on detected portals:

```bash
npm run ats:liveness -- https://boards.greenhouse.io/COMPANY
```

Build a list of 10–15 backup employers. Run monthly — not to apply,
but to maintain a warm list. Public companies (SAP America, Deloitte,
Microsoft) will not appear in Form D data. Absence from Form D is not
a negative signal for these employers.

**Step 4 — Manual lookups (proposed scripts not yet implemented)**

DOL LCA disclosure —
`https://www.dol.gov/agencies/eta/foreign-labor/performance`
Download the most recent quarterly Excel file. Filter by employer
name. Record wage level distribution (% at Level I, II, III, IV) and
SOC codes filed under. Level I/II concentration above 70% warrants
follow-up with HR before petition prep begins.
Proposed: `SCRIPTS/lca/fetch_lca_employer_summary.py`
[DOES NOT EXIST YET]

USCIS H-1B Employer Data Hub —
`https://www.uscis.gov/tools/reports-and-studies/h-1b-employer-data-hub`
Look up employer by name. Record `Total Approvals`, `Total Denials`,
computed `Approval_Rate`. Compare against direct-hire tech benchmark
(~95–98%).
Proposed: `SCRIPTS/h1b/fetch_employer_approval_rate.py`
[DOES NOT EXIST YET]

## Rules

* Read `modes/_shared.md` before running any step.
* Read the `-audit.md` for each dataset before querying it.
* Use only data from engine datasets and manual lookups above.
* Do not treat absence from `SEC_DOL_H1b_data_mapped.csv` as a
  sponsorship problem. State it as a coverage gap.
* Do not infer petition outcome from filing count alone — `Approval_Rate`
  and wage level are separate signals.
* If a result comes from an LLM judgment, label it as such.
* Log every script run and manual lookup in `modes/RUN_LOG.md`.
* Reassess if employer announces layoffs, restructuring, or contract loss.

## What This Mode Cannot Verify

| Gap                                   | Reason                                 |
|---------------------------------------|----------------------------------------|
| Employer H-1B denial rate             | USCIS hub not yet scraped              |
| Wage level distribution of filings    | LCA disclosure not yet ingested        |
| Whether employer will file next cycle | No forward-looking data source exists  |
| Which SOC code employer will use      | Depends on job description             |
| Ashby/Workable ATS detection          | Scrapers not yet implemented           |

## Output

Save report as `data/ats/reports/YYYY-MM-DD-employer-sponsorship-audit.md`

```
# H-1B Sponsorship Audit — YYYY-MM-DD
## Current Employer: ___

### Check 1 — Mapped Dataset
- Record found: YES / NO / AMBIGUOUS
- Total Approvals (if found): ___
- Total Denials (if found): ___
- Approval_Rate (if found): ___
- median_salary_offered (if found): $___
- Data gap note: ___

### Check 2 — BLS Role Resilience
- SOC code: ___
- cognitive_pivot_score: ___
- Resilience: HIGH / MEDIUM / LOW
- annual_median_wage (OEWS 2024): $___

### Check 3 — Backup Employer List
| Company | In Dataset | Form D | ATS | Live Roles | Sponsor Type |
|---------|-----------|--------|-----|------------|--------------|
| ___     | YES/NO    | YES/NO/N/A (public) | Greenhouse/Lever/Not found | LIVE/NONE | DIRECT/CONSULTING |

### Check 4 — Manual Lookups
- LCA wage level: ___% Level I, ___% II, ___% III, ___% IV
- USCIS Approval_Rate: ___%

### Risk Rating
- Sponsorship confidence: HIGH / MEDIUM / LOW
- Backup list readiness: STRONG / PARTIAL / NONE
- Action: ___
- Next audit: YYYY-MM-DD
```

## Log Entry for `modes/RUN_LOG.md`

```
YYYY-MM-DD — H-1B sponsorship audit: EMPLOYER NAME

* Mode: h1b-sponsorship-audit
* Inputs: `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`,
  `data/bls/compact/soc_occupation_compact.csv`,
  `SCRIPTS/ats/detect_ats.py`, `npm run ats:liveness`
* Outputs: `data/ats/reports/YYYY-MM-DD-employer-sponsorship-audit.md`
* Result: ___
* Open issues: SCRIPTS/lca/fetch_lca_employer_summary.py not yet
  implemented — manual DOL lookup required. Ashby not yet supported.
```

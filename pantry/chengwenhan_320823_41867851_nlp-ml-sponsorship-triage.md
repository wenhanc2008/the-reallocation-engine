# Mode: nlp-ml-sponsorship-triage — NLP/ML Role Triage for OPT Students

Use this mode when you are an international CS graduate student with an NLP or
ML focus, on F-1 OPT, and need to quickly separate companies worth applying to
from companies that will waste your application window.

The core problem this mode addresses: ML job postings look similar on the
surface. A posting at a well-funded company with a ten-year H-1B sponsorship
history and a posting at a ghost employer with no funding and no sponsorship
record both say "LLM experience preferred." This mode uses verified data to
separate them before you spend time on applications.

## Required Context

Read first:

- `modes/_shared.md`
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md`
- `data/bls/bls-audit.md`
- `data/bls/compact/soc_occupation_compact.csv`
- `modes/RUN_LOG.md`

## Target SOC Codes for NLP/ML Roles

These are the SOC codes most commonly associated with NLP/ML engineering roles.
Confirm the correct code against `data/bls/compact/soc_occupation_compact.csv`
before filtering.

| SOC Code | Title | Notes |
|---|---|---|
| 15-2051 | Data Scientists | Most NLP research roles map here |
| 15-1252 | Software Developers, Applications | ML engineer roles often coded here |
| 15-1211 | Computer and Information Research Scientists | Research-heavy NLP roles |
| 15-1243 | Database Architects | Occasionally used for data pipeline roles |

If a role title does not map cleanly, note it as unclassified and do not
assign a SOC-based cognitive score.

## Workflow

### Step 1 — Build a Company Shortlist from Verified Data

Start from the SEC/H-1B mapped dataset, not from a job board.

```bash
python3 SCRIPTS/sec/refresh_recent_sec_quarters.py
python3 SCRIPTS/audit_sec_dol_h1b_data.py
```

Filter `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv` for companies
that meet all three of the following criteria:

- `Total Approvals` >= 5 for SOC codes 15-2051, 15-1252, or 15-1211
- `latest_funding_amount` present and funding date within the last 24 months
- `Approval_Rate` >= 0.70

Record the resulting shortlist as
`data/ats/nlp-ml-shortlist-YYYY-MM-DD.csv`.

Do not add companies to the shortlist based on reputation or name recognition
alone. If a company is not in the dataset, mark it as unverified and handle it
separately in Step 5.

### Step 2 — Score Role Resilience Using BLS/O*NET Data

```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py
```

For each SOC code in the shortlist, pull the cognitive demand score from
`data/bls/compact/soc_occupation_compact.csv`.

Report the following fields:

- `soc_code`
- `occupation_title`
- `cognitive_demand_score`
- `automation_risk_flag` if present in the compact table

Roles with `cognitive_demand_score` >= 70 (on the O*NET scale) are considered
resilient for the purposes of this mode. Roles below 60 should be flagged for
review — they may be ML-adjacent rather than ML-core, and their sponsorship
and resilience profiles differ.

If the compact table does not contain a score for a given SOC code, write:
`cognitive_demand_score: missing; BLS extract not current.`

Do not substitute an LLM estimate for a missing score.

### Step 3 — Run ATS Detection on the Shortlist

```bash
cd SCRIPTS/ats
python3 detect_ats.py --csv ../../data/ats/nlp-ml-shortlist-YYYY-MM-DD.csv \
  --company-column company_name \
  --output ../../data/ats/nlp-ml-ats-detection-YYYY-MM-DD.csv
cd ../..
```

Record for each company:

- ATS provider detected (Greenhouse / Lever / Ashby / Unknown)
- Portal URL if available

Companies with no detectable ATS and no public careers page should be
deprioritized. They are harder to track and more likely to use recruiter-only
pipelines that do not favor OPT candidates.

### Step 4 — Check Liveness for Target Postings

When you have specific job URLs from Step 3 or from direct search:

```bash
npm run ats:liveness -- https://example.com/job/12345
```

Or for a batch:

```bash
npm run ats:liveness -- --file data/ats/nlp-ml-job-urls.txt
```

Keep liveness separate from ATS detection in all reports. A company can have
a detected ATS and no currently live postings.

### Step 5 — Handle Unverified Companies Separately

If a company appears in a job posting but is not in the SEC/H-1B mapped
dataset:

1. Note it as `sponsorship: Unknown — not in dataset`.
2. Do not infer sponsorship likelihood from company size, industry, or name.
3. Run ATS detection and liveness checks as normal.
4. Use `oferta.md` for a full individual evaluation if the role is otherwise
   strong.

Do not promote an unverified company into the shortlist.

### Step 6 — Assemble the Triage Table

Combine verified data from Steps 1–5 into a single triage table saved as
`data/ats/reports/nlp-ml-triage-YYYY-MM-DD.md`.

Output format:

```markdown
# NLP/ML Role Triage — YYYY-MM-DD

## Shortlist Summary

| Company | H-1B Approvals (ML SOC) | Approval Rate | Funding Recency | ATS | Live Posting | Cognitive Score | Priority |
|---|---|---|---|---|---|---|---|
| Anthropic | 12 | 0.92 | 2025-Q3 | Greenhouse | Yes | 78 | Pursue |
| [Company B] | 3 | 0.60 | 2023-Q1 | Lever | Yes | 71 | Investigate |
| [Company C] | 0 | N/A | Not found | Unknown | Unknown | 74 | Skip for now |

## Priority Definitions

- **Pursue**: H-1B approvals >= 5 for target SOC, approval rate >= 0.70,
  funding within 24 months, live posting detected.
- **Investigate**: One or two criteria missing or borderline. Worth a deeper
  check with `oferta.md` before deciding.
- **Skip for now**: No H-1B history for target SOC, stale or no funding, no
  live posting, or not in dataset.

## Unverified Companies

| Company | Source | Reason Unverified | Next Action |
|---|---|---|---|

## Missing Data

- ...

## Next Actions

- ...
```

## Proposed New Data Source

### `data/lca-disclosure/lca-disclosure-YYYY.csv`

**Source**: U.S. Department of Labor, Office of Foreign Labor Certification —
LCA (Labor Condition Application) Disclosure Data, published quarterly at
dol.gov/agencies/eta/foreign-labor/performance.

**Why it belongs**: The existing `SEC_DOL_H1b_data_mapped.csv` provides
aggregate H-1B approval and denial counts per company, mapped to broad SOC
codes. It cannot answer a more specific question: has this company ever filed
an LCA for a role titled "NLP Engineer," "Machine Learning Engineer," or
"Research Scientist — Language Models"? LCA disclosure data contains the
exact job title, prevailing wage, worksite city, and SOC code for every
petition — one row per application. Filtering by job title keywords
("NLP", "language model", "LLM", "machine learning") within a target SOC
code gives an NLP/ML student a verified signal that is one level more
specific than aggregate approval counts.

**Data contract classification**: Source Data Layer. Store under
`data/lca-disclosure/` following the same pattern as `data/bls/` and
`data/80-days-to-stay/`. Do not modify the raw downloaded files. Processed
extracts go to `data/lca-disclosure/processed/`.

**Proposed script** (does not exist yet — proposal only):
```bash
python3 SCRIPTS/lca/download_lca_quarters.py --quarters 2025Q3 2025Q4 2026Q1
python3 SCRIPTS/lca/extract_nlp_ml_roles.py \
  --keywords "NLP,language model,LLM,machine learning,large language" \
  --soc 15-2051 15-1252 15-1211 \
  --output data/lca-disclosure/processed/nlp-ml-lca-extract.csv
```

These scripts do not exist in the current repo. They are proposed additions
to `SCRIPTS/lca/` following the same structure as `SCRIPTS/sec/`. Until they
exist, LCA data must be downloaded manually from the DOL website and filtered
by hand — label any figures from manual inspection as inferred, not verified.

**What this adds to the triage table**: a `LCA Job Titles` column showing
verified petition titles for ML roles at each shortlisted company, separate
from the aggregate `H-1B Approvals` column. A company with 5 aggregate
approvals for SOC 15-2051 but zero LCA filings with "NLP" or "LLM" in the
title is a weaker sponsorship signal for this specific student than a company
with 3 aggregate approvals and 3 LCA filings explicitly for "NLP Research
Scientist."

## What This Mode Can Verify

- H-1B sponsorship history for NLP/ML SOC codes, from the mapped SEC/DOL data
- SEC Form D funding recency and amount, from `data/80-days-to-stay/`
- ATS provider presence, using `SCRIPTS/ats/detect_ats.py`
- Job posting liveness, using `npm run ats:liveness`
- O*NET cognitive demand score for target SOC codes, from BLS compact extract

## What This Mode Cannot Verify

- Whether a company will sponsor **your** specific visa situation (OPT → H-1B
  cap-subject). Approval history shows past behavior, not future intent.
- Whether a role is still open if liveness returns ambiguous results (some ATS
  platforms return 200 for expired postings).
- Team size, engineering culture, or role scope. These require external
  research and should be handled after verified triage, using `deep.md`.
- SOC cognitive score for roles the BLS compact extract does not cover. Do not
  substitute an LLM estimate.
- Sponsorship likelihood for companies not in the mapped dataset. Mark as
  Unknown.

## Log Template

```markdown
## YYYY-MM-DD — NLP/ML sponsorship triage

- **Mode:** nlp-ml-sponsorship-triage
- **Inputs:** data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv,
  data/bls/compact/soc_occupation_compact.csv,
  data/ats/nlp-ml-shortlist-YYYY-MM-DD.csv
- **Commands run:** audit_sec_dol_h1b_data.py, detect_ats.py, ats:liveness
- **Outputs:** data/ats/nlp-ml-shortlist-YYYY-MM-DD.csv,
  data/ats/nlp-ml-ats-detection-YYYY-MM-DD.csv,
  data/ats/reports/nlp-ml-triage-YYYY-MM-DD.md
- **Result:** N companies shortlisted; M prioritized Pursue; K flagged
  Investigate; J skipped
- **Open issues:** companies not in dataset; missing BLS scores for SOC X
```

# Mode: data-ai-opt-triage -- Data/AI OPT Sponsorship Triage

## 1. Overview and When To Use

Use this mode when an international Master's student in Data Science, Machine Learning, AI, or Analytics is trying to prioritize job postings under OPT/STEM OPT time pressure.

This mode is designed to answer a narrower question than `scan.md`:

> Among ATS-exposed jobs, which roles are most worth investigating for a Data/AI OPT student based on title relevance and company-level H-1B evidence?

This is not a general job-search mode. It is for students who need to avoid wasting time on roles that look technical but are actually sales, marketing, security, solutions, or management-heavy, or roles at companies with no visible sponsorship evidence.

## 2. Required Context

Read first:

- `modes/_shared.md`
- `modes/scan.md`
- `modes/pipeline.md`
- `modes/oferta.md`
- `modes/RUN_LOG.md`
- `data/ats/portals.example.yml`
- `data/ats/pipeline.md` if it exists
- `data/ats/scan-history.tsv` if it exists
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md`
- `data/bls/BLS-audit.md`

Use these existing data files:

- `data/ats/pipeline.md`
- `data/ats/scan-history.tsv`
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`
- `data/bls/compact/soc_occupation_compact.csv`

## 3. Existing Commands Used

The current repo can run the ATS scan stage and perform manual bash triage.

**Step 1: Execute ATS Scan**
```
export REALLOCATION_ENGINE_PORTALS='data/ats/portals.example.yml'
npm run ats:scan -- --dry-run
npm run ats:scan
```
This verifies ATS-exposed job URLs and produces `data/ats/pipeline.md`. However, it does not verify Data/AI fit or H-1B fit. 

**Step 2: Interim Bash Triage (Existing Workaround)**
Because the dedicated cross-layer script does not yet exist, use these existing bash utilities to filter the pipeline and simulate the triage logic:

*Filter for Data/AI target titles and explicitly remove noise (Sales/Solutions/Security):*
```
grep -iE "data|ai |machine learning| ml |scientist|engineer" data/ats/pipeline.md | grep -ivE "manager|director|sales|solutions|security|marketing|account|resident" > data/ats/data_ai_pure_pipeline.md
```

*Check company-level H-1B evidence against the 80-Days layer:*
```
grep -i "DATABRICKS" data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv | awk -F, '{print "Company: "$1, "| Approvals: "$3, "| Denials: "$4, "| Rate: "$5"%"}'
```

## 4. Proposed Command

This command does **not** currently exist in the repo. It is proposed to automate the manual bash workflow above.

```
python SCRIPTS/ats/data_ai_opt_triage.py \
  --pipeline data/ats/pipeline.md \
  --h1b data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv \
  --bls data/bls/compact/soc_occupation_compact.csv \
  --output data/ats/data-ai-opt-triage.md
```

### Rationale

`scan.md` can find ATS jobs, but it cannot answer whether those jobs are useful for a Data/AI OPT student. The proposed command belongs in the repo because it would automate the connection between the Job-Ops layer, the 80 Days to Stay layer, and the Cognitive Pivot layer.

The proposed command should:

1. Read job rows from `data/ats/pipeline.md`.
2. Classify each job title for Data/AI relevance.
3. Join each company name against `SEC_DOL_H1b_data_mapped.csv`.
4. Report company-level H-1B evidence.
5. Attach SOC/BLS evidence only when the title-to-SOC mapping is explicit or manually confirmed.
6. Write a structured triage report to `data/ats/data-ai-opt-triage.md`.

## 5. Data/AI Title Triage Rules

A role should be marked `Investigate` only if the title is clearly Data/AI relevant.

Target title signals:
- `Data Scientist`
- `Machine Learning Engineer`, `ML Engineer`
- `AI Engineer`, `Data Engineer`, `Big Data Engineer`
- `Applied Scientist`, `Research Scientist`
- `Analytics Engineer`, `AI Platform`

A role should be marked `Manual Review` if it mixes Data/AI language with ambiguous business or customer-facing language.

Manual review signals:
- `Solutions Architect` with Data/AI terms
- `Solutions Engineer` with ML/Data terms
- `Forward Deployed Engineer`
- `AI Platform Architect`

A role should be marked `Skip` if it is primarily outside the Data/AI OPT target.

Skip signals:
- `Sales`, `Account Executive`
- `Marketing`, `Product Marketing`
- `Security`, `Field Engineering`
- `Manager`, `Director`
- `Intern`, `Volunteer`

These rules prevent the mode from treating every role with "AI" or "Engineer" in the title as a Data/AI target.

## 6. H-1B Evidence Rules

For each company, check:
- `company_name`
- `Total Approvals`, `Total Denials`, `Approval_Rate`
- `median_salary_offered`
- `top_job_titles_sponsored`

Use these labels:

| Label | Meaning |
|---|---|
| `Company-level H-1B evidence` | Company appears in the CSV and has non-null H-1B approval fields. |
| `No H-1B evidence found` | Company appears in the CSV but sponsorship fields are blank, or company is not found. |
| `Related sponsored titles` | `top_job_titles_sponsored` includes technical/data-adjacent titles. |
| `No related title evidence` | Sponsorship exists, but sponsored titles do not appear related to Data/AI work. |

Do not claim that a specific job posting sponsors H-1B. The CSV supports company-level evidence, not role-specific sponsorship guarantees.

## 7. SOC/BLS Rules

Use `data/bls/compact/soc_occupation_compact.csv` only when a SOC mapping is explicit or manually justified.

Possible mappings:

| Role family | Possible SOC | Status |
|---|---|---|
| Data Scientist | `15-2051` | title-inferred unless confirmed |
| Software / ML Engineer | `15-1252` | title-inferred unless confirmed |
| Database / Data Engineering | `15-1243` | title-inferred unless confirmed |

If the title is ambiguous, write:
`SOC unknown; title-to-SOC mapping not verified.`
Do not infer SOC from marketing language alone.

## 8. Decision Rules

| Action | Required Evidence |
|---|---|
| `Investigate` | Data/AI target title + company-level H-1B evidence + SOC/BLS checked or explicitly marked title-inferred. |
| `Manual Review` | Mixed title with Data/AI terms + company-level H-1B evidence, but role duties or SOC mapping are ambiguous. |
| `Skip` | Non-Data/AI title, sales/marketing/security/management-heavy title, or no useful sponsorship evidence. |

## 9. What This Mode Can Verify

This mode can verify:
- whether ATS scan produced job URLs;
- whether those URLs were written to `data/ats/pipeline.md`;
- whether a company appears in the SEC/DOL mapped CSV;
- whether company-level H-1B fields are populated;
- whether sponsored title text appears related to Data/AI work;
- whether BLS/SOC data exists for a manually selected SOC code.

## 10. What This Mode Cannot Verify

This mode cannot verify:
- that a specific job posting will sponsor H-1B;
- that the company will sponsor entry-level candidates;
- that an ATS job title maps to a specific SOC code without manual or script-supported classification;
- that a role with "AI" in the title is actually an AI engineering role;
- that company-level sponsorship history applies to this exact team, role, or hiring cycle.

## 11. Output Format

The proposed command should write to `data/ats/data-ai-opt-triage.md`. 

Expected table format:

| ATS Job Title | Company | Title Class | Company H-1B Evidence | Sponsored Title Match | SOC/BLS Evidence | Action |
|---|---|---|---|---|---|---|
| AI Engineer - FDE | Databricks | Target | Company-level H-1B evidence | Technical titles present | SOC title-inferred | Investigate |
| Senior Specialist Solutions Architect - AI & ML | Databricks | Mixed | Company-level H-1B evidence | Related titles present | SOC uncertain | Manual Review |
| Strategic AI/BI Account Executive | Databricks | Skip | Company-level H-1B evidence | Not enough for this role | SOC not checked | Skip |

## 12. Log Template

```
## YYYY-MM-DD -- Data/AI OPT triage

- **Mode:** data-ai-opt-triage
- **Input:** `data/ats/pipeline.md`; `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`; `data/bls/compact/soc_occupation_compact.csv`
- **Existing command run:** `npm run ats:scan` and Bash `grep` simulation filters.
- **Proposed command:** `python SCRIPTS/ats/data_ai_opt_triage.py ...`
- **Output:** `data/ats/data-ai-opt-triage.md`
- **Worked:** ATS scan produced pipeline rows; bash filters successfully isolated target roles; company-level H-1B evidence checked where company names matched.
- **Did not work:** Role-specific sponsorship and exact SOC classification were not verified automatically.
- **Evidence:** row counts, company H-1B fields, sponsored title text, SOC/BLS fields when checked.
- **Next:** Run full `oferta.md` evaluation on `Investigate` and `Manual Review` rows.
```
# Worked Example — nlp-ml-sponsorship-triage

## Scenario

A Northeastern CS graduate student with an NLP focus, eight months into a
12-month OPT, wants to triage a list of five companies where they have seen
ML engineering postings in the past two weeks. The student wants to know which
companies are worth a full application effort this week.

Target companies (from job board search, not yet verified):
- Anthropic
- Cohere
- Mosaic ML (now part of Databricks)
- a Series A startup called "Syntex AI" (fictional, no SEC record)
- a mid-size enterprise software company called "DataVera Inc." (fictional)

## Inputs Used

- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv` — simulated
  based on the dataset structure described in the repo audit
- `data/bls/compact/soc_occupation_compact.csv` — SOC codes 15-2051 and
  15-1252 pulled from BLS compact extract
- ATS detection: simulated `detect_ats.py` output for the five companies
- Liveness: one posting URL per company, simulated `ats:liveness` result

**Note**: This is a simulation. The SEC/H-1B figures below are illustrative
and drawn from publicly known patterns in H-1B filing data. They are not
pulled from the live dataset. All figures should be treated as inferred,
not verified, until the actual scripts are run against the real data.

## Commands Simulated

```bash
python3 SCRIPTS/audit_sec_dol_h1b_data.py
# Output: data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md

python3 SCRIPTS/bls/extract_soc_occupation_table.py
# Output: data/bls/compact/soc_occupation_compact.csv

cd SCRIPTS/ats
python3 detect_ats.py "Anthropic" "Cohere" "Databricks" "Syntex AI" "DataVera Inc." \
  --output ../../data/ats/nlp-ml-ats-detection-2026-06-03.csv
cd ../..

npm run ats:liveness -- --file data/ats/nlp-ml-job-urls.txt
```

## Mode Output

```markdown
# NLP/ML Role Triage — 2026-06-03

## Shortlist Summary

| Company | H-1B Approvals (ML SOC) | Approval Rate | Funding Recency | LCA NLP/ML Titles | ATS | Live Posting | Cognitive Score | Priority |
|---|---|---|---|---|---|---|---|---|
| Anthropic | 28 | 0.96 | 2025-Q4 | Not checked — script not yet built | Greenhouse | Yes | 81 | Pursue |
| Cohere | 11 | 0.88 | 2025-Q2 | Not checked — script not yet built | Lever | Yes | 81 | Pursue |
| Databricks (Mosaic ML) | 47 | 0.91 | 2025-Q3 | Not checked — script not yet built | Greenhouse | Yes | 78 | Pursue |
| Syntex AI | 0 | N/A | Not found in SEC data | N/A — not in dataset | Unknown | Yes — but unverified portal | 78 | Skip for now |
| DataVera Inc. | 2 | 0.50 | 2022-Q1 | Not checked — script not yet built | Ashby | No — posting redirects to homepage | 65 | Skip for now |

## Priority Definitions

- **Pursue**: H-1B approvals >= 5 for target SOC, approval rate >= 0.70,
  funding within 24 months, live posting detected.
- **Investigate**: One or two criteria missing or borderline.
- **Skip for now**: No H-1B history for target SOC, stale funding, no live
  posting, or not in dataset.

## Unverified Companies

| Company | Source | Reason Unverified | Next Action |
|---|---|---|---|
| Syntex AI | LinkedIn job board | Not found in SEC_DOL_H1b_data_mapped.csv | Run oferta.md; check Crunchbase for funding; email recruiter directly to ask about OPT/H-1B before applying |

## Missing Data

- DataVera Inc. cognitive score is 65 (SOC 15-1252); role may be ML-adjacent
  rather than ML-core. Score is below the 70 threshold for resilient
  classification.
- Syntex AI has no H-1B history. Sponsorship status is Unknown, not Likely.
  Do not infer from company size.
- Liveness for Syntex AI posting returned 200 but the page is a generic
  careers landing page, not a specific role. Status recorded as unverified,
  not Live.

## Next Actions

- Run full oferta.md evaluation for Anthropic and Cohere before applying.
- Contact Syntex AI recruiter directly to confirm OPT and H-1B policy before
  spending application time.
- Do not apply to DataVera Inc. this cycle: stale posting, low approval rate,
  funding over 36 months old, and cognitive score below threshold.
```

## What Was Verified vs. Inferred

| Claim | Status | Source |
|---|---|---|
| Anthropic H-1B approval count | Inferred (simulation) | Would be verified from SEC_DOL_H1b_data_mapped.csv |
| Cohere funding recency | Inferred (simulation) | Would be verified from SEC Form D fields |
| DataVera liveness = stale | Inferred (simulation) | Would be verified by `npm run ats:liveness` |
| Syntex AI not in dataset | Verified pattern (simulation) | SEC_DOL_H1b_data_mapped.csv lookup — company absent |
| Cognitive scores for 15-2051 | Inferred from BLS structure | Would be verified from soc_occupation_compact.csv |
| LCA NLP/ML job titles | Not checked | SCRIPTS/lca/ does not exist yet; would require manual DOL download |

## What I Would Change Based on Seeing the Output

**First**: The cognitive score threshold of 70 needs to be validated against
the actual score distribution in `soc_occupation_compact.csv`. I set 70 as
the threshold before seeing the data. If the scores for ML SOC codes cluster
between 75 and 85, a threshold of 70 may be too low to be a useful filter.
After running the BLS extract script, I would inspect the score distribution
and adjust the threshold to the 40th percentile of the target SOC group.

**Second**: The "Syntex AI" case revealed a gap in the mode: it has no
workflow for companies that have live postings but no SEC record at all. The
current mode says "mark as Unknown and handle with oferta.md," but it does not
give the student a signal for how much time to invest before escalating to a
full oferta evaluation. I would add a sub-step in Step 5 that checks
Crunchbase or SEC EDGAR directly for recent Form D filings before deciding
whether to escalate.

**Third**: The DataVera liveness check returned a redirect to a homepage
rather than a 404. The mode currently records this as "stale," but a student
could reasonably interpret a 200 redirect as "live." The mode should be more
explicit: a redirect to a non-role-specific page should be recorded as
`Liveness: Ambiguous — redirect to homepage` rather than `Stale`, and the
student should be instructed to visit the URL manually before making a final
call.

## Run Log Entry

```markdown
## 2026-06-03 — NLP/ML sponsorship triage (simulation)

- **Mode:** nlp-ml-sponsorship-triage
- **Inputs:** SEC_DOL_H1b_data_mapped.csv (simulated), soc_occupation_compact.csv
  (simulated), five company names from job board search
- **Commands run:** audit_sec_dol_h1b_data.py (simulated), detect_ats.py
  (simulated), ats:liveness (simulated)
- **Outputs:** data/ats/reports/nlp-ml-triage-2026-06-03.md (this document)
- **Result:** 3 companies prioritized Pursue; 1 unverified (Syntex AI);
  1 skipped (DataVera). Simulation only — figures not from live data.
- **Open issues:** Cognitive score threshold needs validation against real BLS
  extract; liveness ambiguity for redirect responses needs clearer mode
  guidance; no workflow for companies absent from SEC dataset with live postings.
```

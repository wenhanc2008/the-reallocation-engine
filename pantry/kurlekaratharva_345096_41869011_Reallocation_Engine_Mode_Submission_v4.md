# Reallocation Engine — Mode Design Assignment
**REDACTED | MS Information Systems, Northeastern University | June 2026**

> Rewritten against the actual repository
> (`github.com/nikbearbrown/the-reallocation-engine`). Every path, column,
> command, and number below was checked against the cloned repo. Where the data
> or a script does not exist, this submission says so instead of pretending.

---

# Part 1: Mode File

*Submitted as: `modes/funded-systems-analyst.md`*

---

# Mode: funded-systems-analyst -- ERP / IT-Systems-Analyst Sponsorship Triage at Form D Companies

Use this mode before applying to **IT Business Systems Analyst, ERP / application
support analyst, or computer systems analyst** roles **inside privately funded
companies** (biotech, fintech, deep-tech, SaaS) — not at IT-services
consultancies. Run it once per week before resume tailoring or outreach. Output
decides which employers are worth time that week.

Designed for: an MS Information Systems graduate on F-1 OPT (STEM-extension
eligible, ~36 months runway) with an ERP / ServiceNow / SQL-monitoring /
SLA-incident background, targeting ~$90K–$110K roles at companies that have an
H-1B sponsorship record. The user's skills sit *inside* product companies that
run enterprise systems (e.g. a Series-D biotech running Microsoft Dynamics AX),
not only at the Infosys/Wipro/TCS tier.

**Do not use this mode to look for the big IT-services consultancies.** They do
not file SEC Form D, so they are absent from this engine's company table. If you
want those, you need a different sponsorship dataset the repo does not ship.

## Required Context

Read first:

- `modes/_shared.md` — prime directive and verified-data rules
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv` — 30,369 Form D companies joined to DOL H-1B outcomes
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md` — what columns in that file are reliable
- `data/bls/compact/soc_occupation_compact.csv` — 1,016 occupations with `cognitive_pivot_score`
- `data/bls/bls-audit.md`
- `SCRIPTS/ats/README.md`
- `modes/RUN_LOG.md`

## What This Mode Can Verify

- Whether a company appears in the Form D + H-1B mapped table at all.
- That company's **aggregate** `Total Approvals`, `Total Denials`, `Approval_Rate`
  (lifetime totals in the file — **not** per fiscal year).
- The company's `latest_funding_stage`, `latest_funding_amount`,
  `latest_funding_date` (Form D — the core signal here).
- The free-text `top_job_titles_sponsored` string, when present.
- Whether a known job-posting URL is currently live (`npm run ats:liveness`).
- Whether a company exposes Greenhouse or Lever (`SCRIPTS/ats/detect_ats.py`).
- The `cognitive_pivot_score` and OEWS median wage for a SOC code **that I assign
  by hand** (the data carries no SOC code — see gaps).

## What This Mode Cannot Verify

- **Role-specific sponsorship.** There is no SOC code in the sponsorship file.
  The only role signal is the free-text `top_job_titles_sponsored` column, and it
  is populated for only **1,557 of 30,369 rows (~5%)** — the same 1,557 that have
  any approvals. So a company with a blank title field is *unknown*, not *no*.
- **Per-year or recent sponsorship.** Counts are lifetime aggregates. A 100%
  approval rate over 4 approvals is not the same as a sustained sponsor.
- **New cap-subject intent for an OPT hire** vs. renewals/transfers. No dataset
  here distinguishes them.
- **L1 vs. L2/L3** role level — requires manual JD reading.
- **Salary** for a specific posting — the file has `median_salary_offered` but it
  is sparse; BLS OEWS is a national reference, not the offer.

Do not claim sponsorship from a careers page or a job posting alone.

## Target Occupations (assigned manually, not from the data)

Because the company table has no SOC field, I map *my* target roles to SOC codes
myself and look those up in the BLS compact table:

| SOC | Title | `cognitive_pivot_score` | OEWS median |
|---|---|---|---|
| 15-1211 | Computer Systems Analysts | 4.024 | $103,790 |
| 15-1242 | Database Administrators | 3.958 | $104,620 |
| 15-1244 | Network & Computer Systems Administrators | 3.931 | $96,800 |
| 13-1111 | Management Analysts | 4.013 | $101,190 |
| 15-1232 | Computer User Support Specialists | 3.306 | $60,340 |

(Verified by lookup in `data/bls/compact/soc_occupation_compact.csv` on
2026-06-04. The score scale runs ~1.6–4.9, not 0–10. Codes 15-1299 "Computer
Occupations, All Other" and 15-2051 "Data Scientists" have a **blank** score in
the table — they cannot be scored.)

15-1232 is the L1-helpdesk floor; its low score (3.306) and $60K median are the
"do not anchor here" line.

## Evidence Blocks

### A. Company + sponsorship + funding (one query, real columns)

There is **no** `filter_sponsors_by_soc.py` and **no** `soc_code` column. This is
an inline pandas query against the real file:

```bash
python3 -c "
import csv, re
rows = list(csv.DictReader(open('data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv')))
spons = [r for r in rows if (r['Total Approvals'] or '0').strip() not in ('','0')]
pat = re.compile(r'analyst|business system|systems analyst|erp|application support|service deliver|implementation|consultant|support engineer', re.I)
hits = [r for r in spons if pat.search(r.get('top_job_titles_sponsored',''))]
hits.sort(key=lambda r:(float(r.get('Approval_Rate') or 0), float(r.get('latest_funding_amount') or 0)), reverse=True)
for r in hits[:15]:
    print(r['company_name'],'|',r['latest_funding_stage'],'|',r['latest_funding_date'],'|',
          r['Total Approvals'],'/',r['Total Denials'],'| rate',r['Approval_Rate'],'|',
          r['top_job_titles_sponsored'][:70])
"
```

Report each company as:
- **Title-matched sponsor** — in the 1,557, and `top_job_titles_sponsored`
  contains a systems-analyst / ERP / support title.
- **Sponsor, role unknown** — has approvals but no title text, or only SWE titles.
- **Not found / no approvals** — absent from the file or `Total Approvals` is 0/blank.

### B. Liveness + ATS (real commands)

```bash
# liveness takes URLs:
npm run ats:liveness -- "https://job-url-1" "https://job-url-2"
npm run ats:liveness -- --file data/ats/job-urls.txt
# ATS detection takes COMPANY NAMES, not URLs:
cd SCRIPTS/ats && python3 detect_ats.py "ImmunityBio, Inc." "Lyell Immunopharma"
```

Keep separate: *ATS provider detected* (Greenhouse/Lever = the scrapers the repo
supports; Ashby has a provider stub; Workday/iCIMS are **not** scraped — verify by
hand), *URL live*, *URL stale/closed*, *not checked*.

### C. Role-quality tiebreaker

Look up the SOC code **you assigned** in
`data/bls/compact/soc_occupation_compact.csv`. Use `cognitive_pivot_score` only as
a tiebreaker between comparable live postings. If the SOC for a posting is not
confirmed, write `Cognitive score: not classified — SOC not confirmed.` The table
cannot tell L1 from L2/L3; that is a manual JD read.

### D. CV fit

Map JD requirements to verified CV text only (`resumes/`). Do not invent tools,
certs, or experience. `npm run resumes:pdf -- resumes/<file>.md` generates the PDF.

### E. Recommendation tiers

- **Pursue** — live posting + title-matched sponsor + recent funding + L2/L3 JD.
- **Investigate** — useful but ≥1 field missing/inferred (e.g. sponsor but role unknown).
- **Skip for now** — closed posting, not in the file, 0 approvals, or L1-only framing.

## Output Format

```
# Triage: [Employer] — [Role]
Date: YYYY-MM-DD
Recommendation: Pursue | Investigate | Skip for now

| Area            | Status                                   | Evidence                          |
|-----------------|------------------------------------------|-----------------------------------|
| In company file | Yes / No                                 | row found                         |
| Sponsorship     | Title-matched / Role-unknown / None      | Total Approvals, Total Denials, Approval_Rate (lifetime) |
| Funding (Form D)| Stage / date / amount                    | latest_funding_* fields           |
| ATS             | Greenhouse / Lever / Not-scraped         | detect_ats.py output              |
| Liveness        | Live / Closed / Not checked              | ats:liveness output               |
| Cognitive score | <score> for SOC <code> / Not classified  | cognitive_pivot_score lookup      |
| CV fit          | Strong / Mixed / Weak / Not checked      | matched requirements              |

## Missing data
- ...
## Next action
- ...
```

## Not Implemented Yet (honesty section)

- No `filter_sponsors_by_soc.py` — and it could not be written, because the
  company file has no SOC code. The only honest version is a **title classifier**
  over `top_job_titles_sponsored` (proposed below), not a SOC filter.
- No salary-floor check in this mode. BLS OEWS national medians are referenced
  manually from the table above.
- The repo's own `modes/pipeline.md` already says: *"If SOC is unknown, mark it
  as 'SOC not classified yet.' Do not invent SOC mappings; add a future task for a
  classifier."* This mode follows that rule rather than working around it.

## Proposed New Command (with rationale)

`SCRIPTS/ats/classify_sponsored_titles.py` — read `top_job_titles_sponsored`,
emit a normalized role-family tag (`systems_analyst`, `erp_support`,
`swe`, `data`, `helpdesk`, `other`) per company, and log a coverage number
(how many of the 1,557 it could tag). **Why it belongs:** it converts the one
role signal that exists into a queryable field *and* surfaces its own recall, so
the 95% of companies with no title text stay visibly "unknown" instead of
silently dropping out.

## RUN_LOG.md Template

```markdown
## YYYY-MM-DD — funded-systems-analyst triage

- **Mode:** funded-systems-analyst
- **Inputs:** data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv; data/bls/compact/soc_occupation_compact.csv
- **Command:** inline pandas title filter (Block A); npm run ats:liveness; detect_ats.py
- **Output:** data/ats/reports/YYYY-MM-DD-systems-analyst-triage.md
- **Worked:** companies title-matched: N of 1,557; live postings: N
- **Verified:** in-file presence, aggregate approvals/denials, funding stage/date, liveness, ATS provider, SOC score
- **Inferred:** SOC assignment (manual), role family from title text, L2/L3 level, new-OPT-hire intent, salary floor
- **Did not work / missing:** companies with blank title field = unknown, not negative
- **Next:** ...
```

---
---

# Part 2: Domain Justification

## Who uses this mode and when

An MS Information Systems graduate at a US university, on F-1 OPT starting after a
late-2026 graduation, STEM-extension eligible. Background is functional and
operational — ERP / enterprise-application support, ServiceNow/JIRA incident
management, SQL error monitoring, SLA-driven resolution; not software
engineering. Salary floor ~$90K. Runs the mode weekly, before deciding where to
spend tailoring and outreach effort.

## The information asymmetry

The naive read of an H-1B sponsorship list is **"big total = safe target."** This
engine's data quietly refutes that, and the asymmetry has two layers the student
cannot see without running the mode:

1. **The dataset's universe is Form D-funded private companies, not
   consultancies.** Of 30,369 companies, only **1,557 (~5%)** have *any* H-1B
   approval record, and the public IT-services giants the student is told to chase
   (Infosys, TCS, Wipro, HCL, Capgemini) **are not in the file at all** — they
   don't file Form D. The real, visible sponsors for a systems-analyst profile are
   places like a Series-D biotech running Dynamics AX. A student who never opens
   the file aims at the wrong universe entirely.

2. **There is no role-level sponsorship signal — and the student would assume
   there is.** Sponsorship is keyed to a company, with only a sparse, free-text
   list of titles. A student cannot pivot "this company sponsors *systems
   analysts*" the way they could pivot LCAs by SOC on a public site, because that
   field does not exist here. The honest answer for 95% of companies is
   "sponsors — role unknown," and only the mode makes that gap legible.

## Connection to engine layers

- **80 Days to Stay (primary):** the Form D + H-1B mapped table is the spine.
  Funding stage/date is a real freeze-risk signal here precisely because these are
  private companies — the exact case Form D was built for.
- **Job-Ops:** `ats:liveness` and `detect_ats.py` stop the student tailoring a
  resume for a dead posting; Greenhouse/Lever detection is a funded-headcount hint.
- **Cognitive Pivot:** `cognitive_pivot_score` separates a durable systems-analyst
  role (15-1211 = 4.024) from an L1-helpdesk floor (15-1232 = 3.306, $60K) — used
  as a tiebreaker, with the explicit caveat that the SOC table can't see L-level.

## Failure modes specific to this domain

1. **Silent "unknown→no" collapse on the 95% blank-title rows.** Because only
   1,557 rows carry title text, a title filter quietly hides every company whose
   field is blank. The student concludes "no systems-analyst sponsors here" when
   the truth is "the data can't say." This is hardest to catch for *this* user,
   whose target titles ("IT Business Systems Analyst," "ERP support") are rarer in
   the engineer-skewed title text than "Software Engineer" — so the recall gap
   bites their exact profile worst. Mitigation: report blank-title companies as a
   separate **unknown** bucket, never fold them into the negatives.

2. **Lifetime-aggregate / small-N approval rates read as "safe."** Many top hits
   show `Approval_Rate = 100%` — but over a handful of lifetime approvals with no
   year attached. A student reads 100% as a guarantee and over-invests, when it may
   be one or two old approvals at a company that hasn't sponsored since. No field
   in this file dates or cap-classifies the petitions, so this error is invisible
   from the data alone and only recruiter contact closes it.

---
---

# Part 3: Worked Example

**This output is real.** Every block below was actually executed on **2026-06-04**
— the company filter against `SEC_DOL_H1b_data_mapped.csv`, the SOC lookup against
the BLS compact table, `detect_ats.py` against Greenhouse/Lever over the network,
and the repo's Playwright `check-liveness.mjs`. Numbers and statuses are copied
from command output, not invented.

## Inputs

- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv` (30,369 rows)
- Block A title filter: `analyst|business system|systems analyst|erp|application support|service deliver|implementation|consultant|support engineer`
- `data/bls/compact/soc_occupation_compact.csv` for the role-quality lookup
- `python3 SCRIPTS/ats/detect_ats.py "ImmunityBio, Inc." "Lyell Immunopharma" "Lacework" "Automation Anywhere" "Generate Capital"`
- `npm run ats:liveness -- "<live Lyell URL>" "<dead job id>"`

## Block A — what the query actually returned

```
companies with sponsorship history: 1557  (of 30369)
sponsors with analyst / ERP / support / BI-ish titles: 213
```

Top rows (verbatim from output):

| Company | Stage | Latest funding | Appr/Den | Rate | Title text (matched) |
|---|---|---|---|---|---|
| IMMUNITYBIO INC | Series D+ | 2023-09-11 | 20/0 | 100% | `Application Systems Analyst-Dynamics AX`, Senior Statistical Prog… |
| LYELL IMMUNOPHARMA INC | Series D+ | 2020-03-05 | 8/0 | 100% | `Sr. Business Analyst, Financial Systems` |
| LACEWORK INC | Series D+ | 2021-11-12 | 112/0 | 100% | Software Engineer, UX Researcher, `Staff IT Business Systems Eng…` |
| GENERATE CAPITAL INC | Series D+ | 2021-04-23 | 32/0 | 100% | `Financial and Investment Analyst`, Accountant… |
| AUTOMATION ANYWHERE INC | Series D+ | 2019-11-20 | 164/0 | 100% | `Sr. Technical Support Engineer`, Lead Cloud Ops… |

**The single best real hit: IMMUNITYBIO INC** — a Form D Series-D+ biotech that
sponsored an *"Application Systems Analyst – Dynamics AX"*. That is an ERP /
IT-systems-analyst role at a funded product company — exactly the profile the
draft mode was chasing at consultancies, found where it actually lives.

## Block C — role-quality lookup (real)

Mapping "Application Systems Analyst" → SOC **15-1211 Computer Systems Analysts**:

```
15-1211 | Computer Systems Analysts | cognitive_pivot_score = 4.024 | OEWS median = $103,790
```

Above the $90K floor, and near the top of the score range (~1.6–4.9). Compare the
helpdesk floor I would *skip*: 15-1232, score 3.306, median $60,340.

## Block B — ATS + liveness (real run, 2026-06-04)

ATS detection (`SCRIPTS/ats/detect_ats.py`) run live against the top-5 shortlist:

```
[1/5] ImmunityBio, Inc.   -> none        (0 jobs)   greenhouse 404, lever 404
[2/5] Lyell Immunopharma  -> greenhouse  (11 jobs)  boards.greenhouse.io/lyellimmunopharma
[3/5] Lacework            -> none        (0 jobs)
[4/5] Automation Anywhere -> none        (0 jobs)
[5/5] Generate Capital    -> none        (0 jobs)
```

**This flipped the recommendation.** My single best *sponsorship* hit — ImmunityBio
— is **invisible to the ATS layer**: it is not on Greenhouse or Lever (it runs an
ATS the engine does not scrape, almost certainly Workday or iCIMS). So the engine
cannot collect or liveness-check an ImmunityBio posting; that has to be done by
hand on its careers site. Lyell, by contrast, exposes a Greenhouse board with 11
open jobs.

Fetching Lyell's live board, then running the **real** liveness tool
(`npm run ats:liveness`, Playwright) on a live posting and a fabricated dead job id:

```
✅ active     https://job-boards.greenhouse.io/lyellimmunopharma/jobs/7747232003
❌ expired    https://job-boards.greenhouse.io/lyellimmunopharma/jobs/9999999999
           redirect to https://job-boards.greenhouse.io/lyellimmunopharma?error=true

Results: 1 active  1 expired  0 uncertain
```

The tool works exactly as the mode claims: a dead Greenhouse job returns HTTP 200
but redirects to the board with `?error=true`, and the checker reports `expired`.

**But the second honest finding undercuts Lyell anyway.** All 11 of Lyell's live
jobs are clinical / biostatistics / QA / medical-director roles — **zero**
IT-systems-analyst or business-analyst openings today:

```
Associate Director, Biostatistics · Director, Clinical Operations · Director, Field Medical ·
Director, MSAT · Manager, QA · Manager, Quality Control · Medical Director, Oncology ·
Program Manager II (Contract) · Senior Associate, QA · Sr. Specialist, eQMS · VP, Clinical Development
```

The "Sr. Business Analyst, Financial Systems" that put Lyell on the sponsorship
list was a *historical* petition — it is **not open now**. Sponsorship history is
a lagging signal; the live ATS layer is what tells you whether the role exists
this week.

## Final Output

```
# Triage: Funded-systems-analyst shortlist — run 2026-06-04
Recommendation: Investigate (ImmunityBio, off-engine) | Skip-this-week (Lyell, no IT role live)

| Company        | Sponsorship (lifetime) | Funding (Form D)    | ATS            | Liveness            | SOC score        | Verdict          |
|----------------|------------------------|---------------------|----------------|---------------------|------------------|------------------|
| ImmunityBio    | 20/0, 100%, title-match| Series D+ 2023-09-11| NOT on GH/Lever| can't check (off-engine) | 4.024 (15-1211) | Investigate by hand |
| Lyell Immuno.  | 8/0, 100%, title-match | Series D+ 2020-03-05| Greenhouse, 11 | tool verified active/expired | 4.024 (15-1211) | Skip this week — no IT role live |

## Missing data
- No SOC code in the company file; 15-1211 assigned by hand from the title string.
- 100% approval rates are LIFETIME over small N (8–20), undated — not "current sponsor".
- ImmunityBio's ATS is not scraped by the engine; its postings need manual liveness.
- New-OPT-hire intent not verifiable from any field here.

## Next action
- ImmunityBio: check its careers site by hand for a live Dynamics/systems-analyst posting.
- Lyell: re-run the board weekly; act only when an IT/BSA role appears (history says it can).
- For any live posting found: map the JD to resumes/ for CV fit before applying.
```

## Verified vs. Inferred

| Claim | Status |
|---|---|
| ImmunityBio in the file; 20/0 approvals; Series D+ 2023-09-11 | **Verified** — read from CSV |
| 1,557 of 30,369 companies have approvals; 213 title-match | **Verified** — query output |
| SOC 15-1211 score 4.024, median $103,790 | **Verified** — BLS compact lookup |
| ImmunityBio not on Greenhouse/Lever; Lyell on Greenhouse, 11 jobs | **Verified** — `detect_ats.py` live run |
| Lyell live URL `active`, dead id `expired` (→ `?error=true`) | **Verified** — `check-liveness.mjs` (Playwright) |
| Lyell has no live IT/systems-analyst role today | **Verified** — Greenhouse board fetch |
| The role *is* SOC 15-1211 | **Inferred** — manual title→SOC mapping |
| 100% rate = a reliable current sponsor | **Not verifiable** — lifetime aggregate, undated |
| They'll file a new cap-subject H-1B for an OPT hire | **Not verifiable from any field** |

## What I would change after seeing this

1. **The sponsorship layer and the ATS layer disagree, and that's the real
   product.** My top sponsorship hit (ImmunityBio) can't even be reached by the
   ATS scrapers, and my #2 (Lyell) has the right sponsorship history but no
   matching live role today. The mode should make this three-way state explicit —
   *sponsor + reachable + role-live* — instead of stopping at "sponsor."
2. **The title filter is recall-bound by a 5%-populated field.** 213 of 1,557 is
   probably an *under*-count, because most rows have no title text. The mode must
   show "unknown" companies as their own bucket, and the proposed
   `classify_sponsored_titles.py` should log its coverage so the gap is never hidden.
3. **Approval_Rate needs a guardrail and a date.** Every top hit is 100%/0-denials
   over small lifetime N. I'd add a minimum-approval threshold and a label warning
   that the rate is undated, so "100%" stops reading as a guarantee — and flag that
   the engine cannot liveness-check companies on Workday/iCIMS at all.

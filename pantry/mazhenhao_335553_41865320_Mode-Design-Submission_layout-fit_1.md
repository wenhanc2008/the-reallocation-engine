# The Reallocation Engine — Mode Design Assignment
## layout-fit: SOC Classification & Sponsorship Fit for IC Layout Roles

**Domain:** IC physical-design / memory (DRAM) layout engineering
**Author situation:** International VLSI / IC-layout master's student on
post-completion OPT, intending to stay in the U.S. and targeting U.S. memory and
semiconductor employers (e.g., Micron) for full-custom layout roles that will
require future H-1B sponsorship.

**Reference repo:** https://github.com/nikbearbrown/the-reallocation-engine

This submission contains all three deliverables in one file:

1. **The Mode File** (the deliverable) — written in the style of the existing modes in `modes/`
2. **Domain Justification** (one page)
3. **Worked Example**

…followed by a 5-minute in-class presentation outline.

---

# 1. Mode File — `layout-fit.md`

*(Written to live in `modes/`, in the style of `modes/oferta.md` and `modes/scan.md`.)*

# Mode: layout-fit -- SOC Classification & Sponsorship Fit for IC Layout Roles

Use this mode when an IC physical-design / layout engineer (analog, custom,
mixed-signal, or digital place-and-route) pastes a job posting or company name
and wants to know three things at once:

1. **Which SOC code will this role most plausibly be filed under, and which
   one is it at risk of being mis-filed under?** This single choice drives the
   H-1B prevailing-wage floor and the role-quality signal.
2. **Does the company show verifiable sponsorship and funding evidence?**
3. **Is the posting actually live?**

This mode exists because "Layout Engineer" has **no dedicated SOC/O\*NET
occupation**. The same person can be classified as a Job Zone 4 engineering
occupation or, if the title is read literally, as a Job Zone 3 *drafting*
occupation -- with a large gap in both prevailing wage and cognitive-pivot
signal. A student cannot see that classification from the outside. This mode
makes the spread visible before they apply.

This mode may produce a recommendation, but every recommendation must show
which parts are verified, inferred, or missing.

## Required Context

Read first:

- `modes/_shared.md` (prime directive + verified-data rules)
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`
- `data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped-audit.md`
- `data/BLS/compact/soc_occupation_compact.csv`
  (**Note:** `modes/_shared.md` lists this as `data/bls/...` lower-case. The
  working tree ships it as `data/BLS/...` upper-case. Use the path that exists
  in the current checkout and log the discrepancy; do not assume the lower-case
  path resolves.)
- `resumes/*.md` if a CV-fit check is requested
- `modes/RUN_LOG.md`

## Candidate SOC Set for Layout Roles

These are the occupations a layout/physical-design role realistically maps to.
Values below are read from `data/BLS/compact/soc_occupation_compact.csv`
(OEWS 2024 wages; `cognitive_pivot_score` = O\*NET Level average of
originality, problem sensitivity, deductive & inductive reasoning, critical
thinking, judgment & decision making, complex problem solving, systems
analysis, and systems evaluation). **Always re-read the CSV at run time; do not
trust the cached numbers below if the table has been regenerated.**

| SOC | Title | Job Zone | Median wage | cognitive_pivot_score |
|---|---|---|---|---|
| 17-2072.00 | Electronics Engineers, Except Computer | 4 | $127,590 | 4.069 |
| 17-2061.00 | Computer Hardware Engineers | 4 | $155,020 | 3.999 |
| 17-2071.00 | Electrical Engineers | 4 | $111,910 | 3.861 |
| 17-2199.06 | Microsystems Engineers | 5 | $117,750 | 4.126 |
| 17-3012.00 | Electrical and Electronics Drafters | 3 | $73,720 | 3.25 |

The last row is the **drafter trap**: the literal word "layout" appears in the
drafting occupation, and a literal title match will pull a senior custom-layout
role down into a Job Zone 3 occupation worth ~$54K less in median wage and a
full point lower in cognitive-pivot signal.

## What This Mode Can Verify

- The exact `cognitive_pivot_score`, job zone, employment, and median wage for
  each candidate SOC, pulled directly from the compact table.
- The **classification spread**: the wage-floor and pivot-score gap between the
  best- and worst-case SOC for the same role.
- Whether the target company appears in `SEC_DOL_H1b_data_mapped.csv` and, if
  so, its `Total Approvals`, `Approval_Rate`, `median_salary_offered`,
  `top_job_titles_sponsored`, and SEC funding fields (`latest_funding_amount`,
  `latest_funding_stage`, `latest_funding_date`).
- Whether the posting URL is currently live, via the ATS liveness tools.

## What This Mode Cannot Verify

Be explicit about these gaps in every report:

- **The actual SOC the employer will file.** No script classifies a layout
  posting to a SOC code today (see Proposed Additions). The mode lists
  *candidate* codes and the spread; it does not know which one ends up on the
  LCA. Mark this `inferred`, never `verified`.
- **Whether the role is custom/analog vs. automated digital P&R.** No dataset
  in the repo records process node, EDA tool flow (Cadence Virtuoso, Synopsys,
  Calibre DRC/LVS), or analog-vs-digital split. This must be read from the
  posting text by hand and labeled `inferred from posting`.
- **Company-specific automation posture.** `cognitive_pivot_score` is an
  occupation-level O\*NET average. It is not a measurement of how much *this
  team* leans on automated place-and-route. Do not present the SOC pivot score
  as "this job is AI-safe."
- **Sponsorship for layout specifically.** `top_job_titles_sponsored` is a
  title-string field. A firm that sponsors "Senior Hardware Engineer" but never
  the literal string "Layout Engineer" can read as a non-sponsor. Treat title
  absence as `Unknown`, not `No`.
- **Large public semiconductor employers.** The dataset's *company universe*
  is SEC **Form D** filers (~30K private, funded companies); H-1B is only an
  enrichment column joined onto them (populated for ~5.1% of rows). A firm that
  never files Form D therefore has no row for H-1B to attach to. Most heavy
  layout sponsors -- Micron, NVIDIA, Qualcomm, TI, Broadcom, GlobalFoundries,
  Marvell -- are public, never file Form D, and are **absent entirely**. A
  `not found` here means "outside this dataset's universe," not "does not
  sponsor." Mark it `Not in dataset`, never `No sponsorship`.

## Primary Tools

Existing, maintained commands (verified to exist in the current checkout):

```bash
# 1. Refresh / inspect the SOC compact table (role-quality layer)
python3 SCRIPTS/bls/extract_soc_occupation_table.py
#    -> data/BLS/compact/soc_occupation_compact.csv  (+ bls-audit.md)

# 2. Sponsorship + funding lookup (80 Days layer): grep the mapped CSV
#    by company name; read columns by header, never by position.

# 3. ATS provider + liveness (Job-Ops layer)
cd SCRIPTS/ats
python3 detect_ats.py "<Company Name>" --output ../../data/ats/<slug>-ats-check.csv
cd ../..
npm run ats:liveness -- <posting-url>
```

## Proposed Additions (do not pretend these exist)

1. **`SCRIPTS/bls/classify_layout_role.py` (proposed, not built).** Input: a
   layout posting's title + responsibilities. Output: ranked candidate SOC
   codes with a confidence flag, so the drafter trap is caught automatically
   instead of by manual lookup. Rationale: the central asymmetry of this mode
   is a classification problem and is currently done by eye.
2. **A `pdk_tools` parser field (proposed).** Extract process node and EDA tool
   strings from posting text into a structured column, so analog/custom vs.
   digital P&R can be inferred consistently. Rationale: this is the single
   biggest driver of whether a layout role is judgment-heavy or automatable,
   and no field captures it today.
3. **Public DOL LCA disclosure data (proposed source).** The current
   sponsorship layer is blind to large public chipmakers (Form D covers private
   offerings only). The public DOL H-1B LCA disclosure file does cover them and
   records the SOC code each employer actually filed under. Wiring it in would
   close the single biggest coverage gap for layout job-seekers, whose top
   sponsors are mostly large public firms. Rationale: without it, the mode
   silently under-covers exactly the employers this domain cares about most.

Until #1 exists, SOC selection is a manual, logged judgment call.

## Workflow

1. Extract role facts from the posting only (title, seniority, analog/digital
   cues, EDA tools named, location, comp if stated). Mark anything absent as
   missing.
2. Build the **candidate SOC band** from the compact CSV: list each plausible
   SOC with its job zone, median wage, and pivot score. Compute the spread.
3. Make a logged, explicit `inferred` SOC pick with a one-line reason (e.g.
   "names Virtuoso + analog custom layout -> 17-2072, not 17-3012 drafter").
4. Run the sponsorship lookup; report `Proven / Likely / Unknown` per the
   `_shared.md` rules. Note whether layout-adjacent titles appear in
   `top_job_titles_sponsored`.
5. Run ATS detection + liveness on the posting URL.
6. Produce the report. Recommend `Pursue / Investigate / Skip for now`.
7. Log the run in `modes/RUN_LOG.md`.

## Output Format

```markdown
# Layout-Fit: <Company> -- <Role Title>

**Date:** YYYY-MM-DD
**URL:** ...
**Recommendation:** Pursue | Investigate | Skip for now

## SOC Classification Band  (verified numbers, inferred pick)
| SOC | Title | Job Zone | Median wage | pivot | Likelihood |
|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | best / likely / trap |

**Inferred SOC pick:** <code> -- <one-line reason>
**Classification spread:** wage $<low>-$<high>; pivot <low>-<high>

## Evidence Summary
| Area | Status | Evidence |
|---|---|---|
| Sponsorship | Proven/Likely/Unknown | approvals, rate, median_salary_offered |
| Layout in sponsored titles | Yes/No/Unknown | top_job_titles_sponsored |
| Funding | value/stage/date or Unknown | SEC fields |
| ATS | Detected/Not detected/Not checked | provider |
| Liveness | Live/Stale/Unknown | check-liveness result |
| Role type (analog vs P&R) | inferred from posting | tools/cues named |

## Missing / Unverifiable
- Actual SOC the employer will file (inferred only)
- Process node / EDA flow if posting is silent
- ...

## Next Action
- ...
```

## Log Template

```markdown
## YYYY-MM-DD -- layout-fit: <company> <role>

- **Mode:** layout-fit
- **Inputs:** posting URL/text; data/BLS/compact/soc_occupation_compact.csv; SEC_DOL_H1b_data_mapped.csv
- **SOC band:** best=<code>/<wage>/<pivot> ... trap=17-3012/$73,720/3.25
- **Inferred pick:** <code> -- <reason>
- **Sponsorship:** Proven/Likely/Unknown (evidence)
- **Liveness:** Live/Stale/Unknown
- **Worked:** ...
- **Did not work / missing:** ...
- **Next:** ...
```

## Prompting Rule

Read the CSVs and run the scripts before any judgment. The LLM may explain the
spread, infer the most likely SOC from posting language, and draft the report.
It must not invent pivot scores, wages, approval counts, sponsorship claims, or
a SOC classification presented as fact. The SOC pick is always labeled
`inferred` until a classifier script produces it.

---

## 2. Domain Justification (one page)

**Who uses this and when.** An international full-custom / memory-layout
engineer on OPT, reviewing U.S. semiconductor postings and deciding which are
worth a sponsorship-dependent application — used *before* applying, to price the
role's wage floor and check the employer's sponsorship realism.

**The information asymmetry it addresses.** "Layout Engineer" / "Physical
Design Engineer" has **no dedicated SOC or O\*NET occupation code.** The same
person legitimately maps to several codes, and the choice — invisible from the
posting — sets the H-1B prevailing-wage floor and signals the role's place in
the Cognitive Pivot thesis:

| If filed as | SOC | Job Zone | Median wage | cognitive_pivot_score |
|---|---|---|---|---|
| Engineering | 17-2061 Computer Hardware Engineers | 4 | $155,020 | 3.999 |
| Engineering | 17-2072 Electronics Engineers | 4 | $127,590 | 4.069 |
| Engineering | 17-2071 Electrical Engineers | 4 | $111,910 | 3.861 |
| **Drafting** | **17-3012 Electrical & Electronics Drafters** | **3** | **$73,720** | **3.25** |

(Values read from `data/BLS/compact/soc_occupation_compact.csv`, OEWS 2024.)

The literal word "layout" sits inside the *drafting* occupation, so a careless
title match drops a judgment-heavy custom-layout role into a Job Zone 3 code
worth ~$54K less and a full point lower in cognitive-pivot signal. Even within
the engineer codes the floor swings ~$43K (17-2071 vs 17-2061). A student cannot
see, from a careers page, which SOC an employer's counsel will file. The mode
makes the spread explicit so the engineer can raise it in screening rather than
discover it on the LCA.

**How it connects to the three engine layers.**
- **Cognitive Pivot** (core): reads `cognitive_pivot_score` from the compact SOC
  table; shows how classification, not just the job, moves the AI-resilience
  signal for the same worker.
- **80 Days to Stay**: sponsorship/funding realism from
  `SEC_DOL_H1b_data_mapped.csv`.
- **Job-Ops**: ATS liveness via `SCRIPTS/ats/`.

**Failure modes (domain-specific, not "the model might hallucinate").**
1. **Drafter mis-pull.** A senior analog/custom layout role gets matched to
   17-3012 on the word "layout," understating wage floor and AI-resilience.
   Hardest to catch for a *junior* layout engineer, who also calls their work
   "layout" and assumes the drafter mapping is simply correct.
2. **Public-employer coverage gap.** The dataset's company universe is SEC
   **Form D** filers (~30K funded private companies); H-1B is only an enrichment
   join, populated for ~5.1% of rows. A firm that never files Form D has no row
   for H-1B to attach to. Checked directly: of 11 major public semiconductor
   H-1B sponsors, 9 are absent (NVIDIA, Qualcomm, TI, Broadcom, Applied
   Materials, GlobalFoundries, Marvell, Analog Devices, Micron); AMD's apparent
   hit is a different company; only Intel appears -- and its funding tag reads a
   nonsensical "$25M Series B" for a firm with ~13,318 H-1B approvals. So a
   `not found` reads like "no sponsorship" when the truth is "outside coverage,"
   and even a public firm that sneaks in carries a garbage funding signal.
   Hardest to catch for a student who trusts the engine's coverage as complete
   and silently drops the best employers. (Micron alone filed 573 LCAs in FY2025
   at ~98% approval -- and is not in the dataset.)
3. **Pivot-score over-trust.** `cognitive_pivot_score` is an occupation average,
   not a measure of *this* team's reliance on automated place-and-route. Reading
   the SOC number as "this specific job is AI-safe" is the error. Hardest to
   catch for someone optimizing purely on the number, because it is real and
   looks authoritative.

---

## 3. Worked Example

**Honesty note.** SOC codes, job zones, wages, and pivot scores are **verified**
— pulled live from `data/BLS/compact/soc_occupation_compact.csv`. The Micron
sponsorship figures are **verified from public DOL LCA disclosure data
(FY2025)**, which is *outside* the engine (see Proposed Addition #3). Whether
the engine *itself* covers Micron was checked directly and is reported honestly.

### Inputs used
- **Role profile (verified from a real posting):** a memory-chip (DRAM)
  "IC Layout Design" campus posting (originally titled in Chinese; translated
  here). Responsibilities: memory-chip
  layout; standard-cell, module, and top-level place-and-route; DRC/ERC/LVS and
  layout review through tapeout. Qualifications: Cadence/Calibre CAD and
  verification tools; ESD / latch-up principles; circuit-design fundamentals.
  This is **full-custom memory layout** — judgment-and-craft heavy, not
  commodity drafting.
- **Target company (U.S. equivalent employer):** Micron Technology, Inc.
  (Boise, ID) — the closest U.S. memory (DRAM) maker for this exact role.
- **Data read:** `soc_occupation_compact.csv` (repo, verified);
  `SEC_DOL_H1b_data_mapped.csv` (repo); public DOL LCA disclosure for Micron
  (external, labeled).

### What the mode produced

```
# Layout-Fit: Micron Technology, Inc. -- Memory (DRAM) Layout Engineer
Recommendation: Pursue (sponsorship strong; price the SOC question in screening)

## SOC Classification Band  (numbers verified; pick inferred)
| SOC        | Title                          | JZ | Median   | pivot | Likelihood |
| 17-2072.00 | Electronics Engineers          | 4  | $127,590 | 4.069 | likely      |
| 17-2061.00 | Computer Hardware Engineers    | 4  | $155,020 | 3.999 | possible    |
| 17-2071.00 | Electrical Engineers           | 4  | $111,910 | 3.861 | employer-used |
| 17-3012.00 | Electrical & Electronics Draft | 3  | $73,720  | 3.25  | TRAP (reject) |

Inferred SOC pick: 17-207x engineer tier -- posting names DRC/ERC/LVS, custom
memory layout, Cadence/Calibre + circuit fundamentals = circuit-design judgment,
not drafting. Reject 17-3012.
Classification spread: wage $73,720-$155,020; pivot 3.25-4.069.

## Evidence Summary
| Sponsorship (engine)       | Not in dataset -- Micron is public, no Form D     |
| Sponsorship (public DOL)   | Strong: 573 LCAs FY2025, ~98% approval            |
| SOC employer actually files| 17-2071 Electrical Engineers (per public LCA)     |
| ATS / Liveness             | Not run (campus posting, not an ATS URL)          |
| Role type                  | inferred from posting: full-custom DRAM (judgment-heavy) |

## Missing / Unverifiable
- The engine cannot see Micron at all (Form D blind spot)
- Which SOC ends up on *your* LCA (employer historically uses 17-2071, low end)
- Process node / specific PDK if posting is silent
```

### What was verified vs. inferred
- **Verified (repo):** all four SOC rows — codes, job zones, median wages, pivot
  scores. The $73,720–$155,020 spread is real.
- **Verified (public DOL, external):** Micron filed 573 LCAs in FY2025 (~98%
  approval) and files semiconductor design engineers under **17-2071 Electrical
  Engineers** — confirming engineer tier, not drafter.
- **Verified (engine check):** Micron is **absent** from
  `SEC_DOL_H1b_data_mapped.csv` (grep returned no match) — the coverage gap is
  real, not an assumption.
- **Inferred:** the engineer-tier SOC read from posting language; the
  full-custom-vs-P&R judgment; the recommendation tier.

### What I would change after seeing the output
1. **Add public DOL LCA disclosure as a data source (Proposed Addition #3).**
   The worked example proved the engine is blind to Micron — the single most
   relevant employer — purely because Micron is public. This is the highest-value
   fix: it both restores coverage and supplies the SOC the employer actually
   files (here, 17-2071).
2. **Surface a screening question as an output field:** "Which SOC code will
   this role be filed under on the LCA?" Micron's own filings sit at 17-2071
   ($111,910, pivot 3.861) — the *low end* of the engineer band — so the spread
   matters even after the drafter trap is rejected. That question is worth more
   than the recommendation tier.
3. Drop ATS/liveness for campus postings (no ATS URL); keep it for U.S.
   careers-page roles where it applies.

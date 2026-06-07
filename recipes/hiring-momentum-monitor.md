# Recipe: hiring-momentum-monitor -- Market And Funding Signals For Job Search

## Executive Summary

Adapted from market momentum and funding intelligence. Use this recipe to find
companies and sectors that may be hiring because of funding, product launches,
new offices, partnerships, government contracts, or rapid category growth.

This recipe helps decide where to search next. It does not claim that a company
has open roles unless job postings are separately verified.

## Required Reads

Read first:

- `recipes/scan.md`
- `recipes/patterns.md`
- `recipes/tracker.md`
- `data/ats/scan-history.tsv` if it exists
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Sector or role lane | Text/JSON | `[TODO: DEFINE] target sector, SOC, or role family` | Yes |
| Momentum sources | RSS/API/local export | `[TODO: DATA SOURCE] funding, partnership, product-launch, office-expansion, or contract sources` | Yes |
| Job posting verification | ATS scan output | `data/ats/` scan outputs | No |

## Phase Gates

1. **Lane gate:** Target role lane or sector is explicit.
2. **Source gate:** Momentum sources are approved and current.
3. **Verification gate:** Momentum signals are not treated as jobs until ATS or
   posting evidence exists.
4. **Small-run gate:** Run on one sector before broad expansion.
5. **Logging gate:** Log any new company targets added to the job-search system.

## Workflow

1. Define sector or role lane.
2. Collect momentum signals.
3. Normalize records into company, signal type, date, source, and relevance.
4. Match companies against existing tracker or scan history.
5. Mark companies as new targets, already tracked, or not relevant.
6. Human chooses which companies enter `scan`, `deep`, or `contacto`.

## Output

Report columns:

- company;
- sector;
- signal type;
- signal date;
- source URL;
- why it may imply hiring;
- whether ATS/job evidence exists;
- recommended next recipe.

Suggested output path:

`reports/generated/hiring-momentum-monitor-[DATE].md`

## Stop Conditions

- Stop if role lane is undefined.
- Stop if funding or product news is presented as verified hiring.
- Stop if source URLs are missing.
- Stop if new targets would be added to tracker without human approval.

## Adapted From

- Madison market momentum funding monitor
- Mycroft funding intelligence agent

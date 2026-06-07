# Recipe: sector-health-dashboard -- Job Market Category Sentiment

## Executive Summary

Adapted from category sentiment dashboards and market sentiment analysis. Use
this recipe to decide whether a job-search sector is expanding, cooling,
unstable, or noisy based on public signals, job postings, funding, layoffs, and
news sentiment.

This supports sector selection and prioritization. It does not replace role-level
posting verification.

## Required Reads

Read first:

- `recipes/patterns.md`
- `recipes/scan.md`
- `recipes/deep.md`
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Sector list | JSON/Markdown | `[TODO: DEFINE] approved sectors or SOC/role families` | Yes |
| Sector signals | JSON/CSV/RSS | `[TODO: DATA SOURCE] postings, layoffs, funding, news, BLS, or SEC/H-1B mapped data` | Yes |
| Dashboard schema | JSON/Markdown | `[TODO: DEV] define fields, date buckets, and score formulas` | Yes |

## Phase Gates

1. **Sector gate:** Sector list is explicit.
2. **Data gate:** Each signal source is cited and current.
3. **Schema gate:** Dashboard fields and score formulas are defined.
4. **Interpretation gate:** Human reviews before changing job-search strategy.

## Workflow

1. Define sectors or role families.
2. Collect job postings and external sector signals.
3. Normalize records by sector, date, signal type, and source.
4. Aggregate into trend buckets.
5. Produce sector health classifications.
6. Human chooses which sectors to prioritize, monitor, or pause.

## Output

Report sections:

- sector list;
- signal coverage;
- job posting trend;
- funding/growth signal;
- layoff/risk signal;
- sentiment summary;
- recommended job-search priority.

Suggested output path:

`reports/generated/sector-health-dashboard-[DATE].md`

## Stop Conditions

- Stop if sector definitions are vague.
- Stop if dashboard formulas are missing.
- Stop if the report recommends sector shifts without showing evidence.
- Stop if old data is mixed with current data without date labels.

## Adapted From

- Madison category sentiment dashboard
- Mycroft market sentiment analysis

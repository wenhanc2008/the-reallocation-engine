# Recipe: employer-reputation-monitor -- Job Search Reputation Signals

## Executive Summary

Adapted from company and brand news intelligence. Use this recipe to monitor
target employers for layoffs, growth, leadership changes, public controversy,
product launches, funding, and hiring news before a student invests time in an
application or interview process.

This is a job-search evidence workflow, not an employer-ranking oracle. It helps
decide whether a company needs deeper review, outreach, caution, or removal from
the target list.

## Required Reads

Read first:

- `recipes/_shared.md`
- `recipes/scan.md`
- `recipes/tracker.md`
- `data/ats/applications.md` if it exists
- `data/ats/scan-history.tsv` if it exists
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Target company list | Markdown/CSV | `data/ats/applications.md`, `data/ats/pipeline.md`, or `[TODO: DATA SOURCE] approved company list` | Yes |
| Employer news sources | RSS/API/local export | `[TODO: DATA SOURCE] approved news, company blog, layoff, funding, and product-launch sources` | Yes |
| Reputation rubric | JSON/Markdown | `[TODO: DEFINE] define growth, risk, controversy, hiring, and stability signals` | Yes |

## Phase Gates

1. **Target gate:** Company list is explicit and tied to the current job-search
   pipeline.
2. **Source gate:** Every source is approved and no credentials are embedded in
   tracked files.
3. **Sample gate:** Run on three companies before broad monitoring.
4. **Evidence gate:** Every risk or opportunity flag links to a source URL or
   local evidence row.
5. **Human judgment gate:** A human decides whether the signal changes job-search
   behavior.

## Workflow

1. Read target companies from the tracker or approved list.
2. Collect recent employer signals from approved sources.
3. Normalize each signal into company, signal type, date, source, URL, and notes.
4. Score signals against the reputation rubric.
5. Add flags to a local handoff report; do not update the tracker automatically.
6. Human reviews flagged companies and decides whether to apply, pause, research,
   or skip.

## Output

Write a report with:

- company;
- role or target lane if known;
- signal type;
- signal date;
- source URL;
- confidence or source reliability;
- suggested job-search action;
- rows needing human review.

Suggested output path:

`reports/generated/employer-reputation-monitor-[DATE].md`

## Stop Conditions

- Stop if company names are ambiguous or unmatched.
- Stop if signals lack source URLs.
- Stop if the workflow tries to infer layoffs, sponsorship, or culture from a
  single weak source.
- Stop before editing application status without human approval.

## Adapted From

- Madison brand/news reputation monitor
- Mycroft news intelligence patterns

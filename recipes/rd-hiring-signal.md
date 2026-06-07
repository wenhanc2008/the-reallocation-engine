# Recipe: rd-hiring-signal -- Patent And Innovation Signals For Job Search

## Executive Summary

Adapted from innovation positioning and patent velocity tracking. Use this
recipe to identify companies investing in AI, cloud, robotics, cybersecurity,
biotech, semiconductors, or other R&D-heavy areas that may imply future hiring
needs.

Patent and innovation activity is a weak hiring signal by itself. Use this only
as an input to deeper company research and ATS verification.

## Required Reads

Read first:

- `recipes/deep.md`
- `recipes/scan.md`
- `recipes/tracker.md`
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Company or sector watchlist | JSON/Markdown | `[TODO: DATA SOURCE] approved company or technology-area list` | Yes |
| Patent or innovation records | JSON/API/local export | `[TODO: DATA SOURCE] USPTO, Google Patents, Lens, company R&D news, or local export` | Yes |
| Hiring relevance rubric | JSON/Markdown | `[TODO: DEFINE] define why an innovation signal matters for target roles` | Yes |

## Phase Gates

1. **Watchlist gate:** Companies or sectors are approved.
2. **Source gate:** Patent/innovation source is approved and cited.
3. **Relevance gate:** Innovation signals must map to target role families.
4. **Verification gate:** ATS/job evidence is checked before adding application
   targets.

## Workflow

1. Load company or sector watchlist.
2. Collect patent and innovation records.
3. Normalize into company, technology area, date, title, source, and assignee.
4. Score relevance to target roles.
5. Cross-check whether the company has relevant open roles.
6. Recommend scan/deep/tracker next steps.

## Output

Report columns:

- company;
- technology area;
- patent or innovation signal;
- date;
- source URL;
- target-role relevance;
- open-role evidence;
- recommended next action.

Suggested output path:

`reports/generated/rd-hiring-signal-[DATE].md`

## Stop Conditions

- Stop if assignee/company mapping is uncertain.
- Stop if patent activity is treated as proof of immediate hiring.
- Stop if target-role relevance rubric is missing.
- Stop if source URLs are missing.

## Adapted From

- Madison innovation positioning patent tracker
- Mycroft patent filing velocity tracker

# Recipe: employer-risk-index -- Job Stability And Trust Signals

## Executive Summary

Adapted from consumer trust/anxiety and brand risk recipes. Use this recipe to
flag employer risks that may affect job-search decisions: public backlash,
regulatory pressure, customer trust erosion, layoffs, product failures, service
quality problems, or leadership instability.

This is a caution workflow. It should not blacklist companies automatically.

## Required Reads

Read first:

- `recipes/deep.md`
- `recipes/tracker.md`
- `data/ats/applications.md` if it exists
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Target employers | Markdown/CSV | Tracker, pipeline, or approved target list | Yes |
| Risk signal sources | RSS/API/local export | `[TODO: DATA SOURCE] layoffs, regulatory, customer trust, incident, or reputation sources` | Yes |
| Risk rubric | JSON/Markdown | `[TODO: DEFINE] define severity, recency, source reliability, and job-search impact` | Yes |

## Phase Gates

1. **Target gate:** Employer list is approved.
2. **Risk rubric gate:** Risk categories and severity are defined.
3. **Evidence gate:** Every risk flag cites a source.
4. **Human gate:** Human decides whether risk changes status or next action.

## Workflow

1. Load target employers.
2. Collect risk signals from approved sources.
3. Normalize into company, risk type, date, severity, source, and evidence.
4. Score risk using the rubric.
5. Compare against current application/tracker status.
6. Recommend keep, monitor, research, pause, or skip.

## Output

Report columns:

- company;
- current application status if known;
- risk type;
- severity;
- evidence URL;
- job-search impact;
- recommended next action;
- human review flag.

Suggested output path:

`reports/generated/employer-risk-index-[DATE].md`

## Stop Conditions

- Stop if risk rubric is missing.
- Stop if evidence URLs are missing.
- Stop if a company is marked skip automatically without human review.
- Stop if private employee/customer data appears in source records.

## Adapted From

- Madison consumer trust anxiety index
- Mycroft retail investor anxiety index

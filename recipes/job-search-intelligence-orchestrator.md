# Recipe: job-search-intelligence-orchestrator -- Job Ops Routing

## Executive Summary

Adapted from marketing and financial intelligence orchestrators. Use this recipe
to route a job-search question to the right Reallocation Engine workflow:
scan, tracker, deep research, employer reputation, skill demand, risk, outreach,
apply, interview prep, or training.

This recipe coordinates work. It should not skip phase gates or run live
actions automatically.

## Required Reads

Read first:

- `recipes/_shared.md`
- `recipes/scan.md`
- `recipes/tracker.md`
- `recipes/deep.md`
- `recipes/apply.md`
- `recipes/contacto.md`
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Job-search request | Text | Human request or conductor message | Yes |
| Recipe registry | Markdown/JSON | `recipes/README.md` plus `[TODO: DEV] machine-readable recipe registry` | Yes |
| Current tracker state | Markdown/CSV | `data/ats/applications.md` if it exists | No |

## Phase Gates

1. **Intent gate:** Request is classified as search, research, apply, outreach,
   interview, training, or tracking.
2. **Evidence gate:** Existing tracker and scan data are checked before new work.
3. **Routing gate:** Recommended next recipe is explicit.
4. **Safety gate:** No apply, email, or external write runs without human
   approval.

## Workflow

1. Read the job-search request.
2. Check existing tracker and scan data.
3. Classify intent.
4. Route to the best recipe or sequence of recipes.
5. Identify blockers and missing inputs.
6. Produce next commands and human decisions required.

## Output

Report sections:

- request summary;
- current evidence checked;
- intent classification;
- recommended recipe path;
- blocked gates;
- missing inputs;
- next commands.

Suggested output path:

`reports/generated/job-search-intelligence-orchestrator-[DATE].md`

## Stop Conditions

- Stop if request intent is ambiguous.
- Stop if the orchestrator would skip local evidence checks.
- Stop if it attempts to apply, email, or update trackers without approval.
- Stop if no existing recipe fits and the gap is not stated.

## Adapted From

- Madison marketing intelligence orchestrator
- Mycroft orchestrator recipes

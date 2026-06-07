# Recipe: target-company-comparison -- Employer Fit Comparison

## Executive Summary

Adapted from competitive positioning and comparative analysis recipes. Use this
recipe to compare target employers by role fit, tech stack, hiring momentum,
public reputation, sponsorship clues, location constraints, and learning value.

This recipe supports a ranked target-company shortlist. It does not decide for
the student; it shows tradeoffs.

## Required Reads

Read first:

- `recipes/_shared.md`
- `recipes/deep.md`
- `recipes/tracker.md`
- `data/ats/applications.md` if it exists
- `data/ats/scan-history.tsv` if it exists
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Company shortlist | Markdown/CSV | `data/ats/applications.md` or `[TODO: DATA SOURCE] approved shortlist` | Yes |
| Job posting evidence | Markdown/CSV/HTML | ATS scan outputs or saved postings | Yes |
| Fit rubric | Markdown/JSON | `[TODO: DEFINE] define scoring for skills, sponsorship, location, growth, risk, and mission fit` | Yes |
| Company research | Markdown/JSON | employer reputation, funding, product, and tech-stack outputs | No |

## Phase Gates

1. **Shortlist gate:** Companies are explicitly named.
2. **Evidence gate:** Every comparison criterion has supporting evidence or is
   marked unknown.
3. **Rubric gate:** The fit rubric is defined before scoring.
4. **No-fiction gate:** Unknown sponsorship, salary, or culture fields stay
   unknown.
5. **Review gate:** Human confirms the ranking before using it for outreach.

## Workflow

1. Load target companies and roles.
2. Pull job posting evidence from scan/tracker outputs.
3. Pull optional company research signals.
4. Score each company against the fit rubric.
5. Produce a comparison table and notes for unknown fields.
6. Human chooses which companies move to apply, outreach, deep research, or skip.

## Output

Report columns:

- company;
- role;
- URL;
- skill fit;
- sponsorship evidence;
- location fit;
- hiring momentum;
- employer risk;
- learning value;
- recommended next action;
- evidence links.

Suggested output path:

`reports/generated/target-company-comparison-[DATE].md`

## Stop Conditions

- Stop if the rubric is missing.
- Stop if company evidence is too thin to compare honestly.
- Stop if the output ranks companies without showing unknown fields.
- Stop if recommendations are treated as facts rather than decision support.

## Adapted From

- Madison competitive positioning agent
- Mycroft comparative analysis agent

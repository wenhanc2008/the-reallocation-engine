# Recipe: skill-demand-monitor -- Tool And Stack Signals For Job Intelligence

## Executive Summary

Adapted from martech/product positioning and tech-stack signal recipes. Use this
recipe to infer which tools, platforms, and technical skills are appearing in
target-company materials and job postings, then map them to resume keywords,
training priorities, and interview preparation.

This recipe helps prioritize learning and resume language. It does not invent
experience the student does not have.

## Required Reads

Read first:

- `recipes/training.md`
- `recipes/interview-prep.md`
- `recipes/apply.md`
- `data/ats/applications.md` if it exists
- `logs/RUN_LOG.md`

## Inputs

| Input | Type | Source | Required? |
|---|---|---|---|
| Job postings | Markdown/CSV/HTML | ATS scan outputs or saved postings | Yes |
| Company/product sources | Markdown/RSS/API | `[TODO: DATA SOURCE] company docs, engineering blogs, product pages, or release notes` | No |
| Skill taxonomy | JSON/Markdown | `[TODO: DEFINE] map tools to skill categories and resume labels` | Yes |

## Phase Gates

1. **Posting gate:** Job postings are current and source URLs are preserved.
2. **Taxonomy gate:** Tool and skill categories are defined.
3. **Truth gate:** Resume recommendations separate existing skills from learning
   targets.
4. **Training gate:** Human approves any training plan before schedule changes.

## Workflow

1. Extract tools, platforms, frameworks, and certifications from job postings.
2. Optionally enrich with company docs and product pages.
3. Normalize mentions into skill category, tool name, evidence source, and count.
4. Compare extracted skills against the student's profile.
5. Produce resume keyword suggestions and training priorities.
6. Human approves which skills move to resume, project, or training work.

## Output

Report sections:

- top skills by frequency;
- skills by target company;
- existing student strengths;
- gaps to learn;
- resume keywords supported by evidence;
- interview prep topics;
- training recommendations.

Suggested output path:

`reports/generated/skill-demand-monitor-[DATE].md`

## Stop Conditions

- Stop if skill taxonomy is missing.
- Stop if the report recommends adding skills not supported by student evidence.
- Stop if job postings are stale or lack URLs.
- Stop if training recommendations cannot be tied to target roles.

## Adapted From

- Madison martech product positioning signal agent
- Mycroft tech stack directional signal agent

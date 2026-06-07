# Domain Justification — nlp-ml-sponsorship-triage

## Who Uses This Mode and When

An international CS graduate student at Northeastern University, specializing
in NLP or ML, on F-1 OPT with 12–29 months remaining. They have completed
coursework and possibly a research project involving fine-tuning or deploying
large language models. They are entering the job market for ML engineering or
applied research roles and need to allocate their application effort efficiently
before their OPT window closes.

The situation is time-constrained in a specific way: OPT candidates cannot
afford to spend two weeks on a thoughtful application to a company that has
never sponsored an H-1B for a software or data science role. The cost of a
missed application is not just the time — it is also the opportunity cost of
not having applied to a company with a strong sponsorship record during the
same window.

## Information Asymmetry Addressed

The asymmetry is this: NLP/ML job postings do not disclose sponsorship
willingness or company financial health. A posting on LinkedIn or a company
careers page looks identical whether the company has sponsored 40 H-1B
petitions in the last three years or zero. A student reading job boards has no
reliable signal to separate these cases without manual research that takes
hours per company.

The engine resolves this asymmetry by combining three verified layers:

**80 Days to Stay layer**: The SEC/DOL H-1B mapped dataset provides actual
approval and denial counts by company and SOC code. This converts "does this
company sponsor?" from a guess into a documented history. For NLP/ML roles
specifically, filtering by SOC codes 15-2051 and 15-1252 narrows the signal to
sponsorship of the exact role type the student is pursuing, not just any role
at the company.

**Job-Ops layer**: ATS detection and liveness checking resolves whether a
posting is real and reachable. Ghost postings — live on LinkedIn but already
filled, or posted by recruiters for roles that no longer exist — are a known
problem in ML hiring. A student who applies to ten stale postings has spent
real time for zero pipeline entries. The liveness check turns this from an
assumption into a verified status.

**Cognitive Pivot layer**: The O*NET cognitive demand score for NLP/ML SOC
codes addresses a longer-horizon question: is this role category likely to
remain a viable career track, or is it among the roles where AI substitution
risk is highest? For an OPT student who is also planning an H-1B cap lottery
cycle (typically 12–18 months after starting OPT), a role with low cognitive
demand score may not be a stable investment of their application window.

## Failure Modes

**Failure Mode 1 — Sponsorship history does not equal sponsorship intent.**
The H-1B approval history in the dataset reflects past petitions, not current
HR policy. A company that sponsored 15 ML engineers between 2019 and 2022 may
have frozen international hiring in 2024. The mode's triage table will still
show this company as high-priority based on its historical record. An OPT
student who does not verify current policy before investing application effort
may waste their strongest applications. This failure is hardest to catch for
students who are new to the US job market and treat historical approval rates
as guarantees. The mode names this gap explicitly in the "cannot verify"
section, but a student under deadline pressure may skip that section.

**Failure Mode 2 — SOC code misclassification creates silent score errors.**
Many NLP/ML engineering job postings are classified under SOC 15-1252
(Software Developers, Applications) in H-1B filings, not 15-2051 (Data
Scientists), even when the actual work is NLP research. If a student filters
on 15-2051 only, they will undercount sponsorship history for companies whose
HR department codes ML roles as software development. The mode lists both SOC
codes, but a student who uses only one will see a misleadingly low approval
count for companies that are actually strong sponsors. This error is hard to
catch because the triage table looks complete — it has numbers in every cell
— but those numbers reflect only one SOC slice of the company's actual
sponsorship activity.

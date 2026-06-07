# ml-sponsorship-triage.md

**Mode:** `ml-sponsorship-triage`
**Status:** Draft (proposed scripts noted; existing scripts confirmed)
**For:** International MS student in CS/Data Science targeting ML Engineering roles, with OPT expiring within 12 months

---

## What This Mode Does

This mode filters the 30K-company sponsorship dataset down to a **ML Engineering-specific shortlist** ranked by five verified or data-grounded signals:

1. H-1B sponsorship history for ML-adjacent SOC codes
2. Recent funding recency from SEC Form D filings
3. Cognitive-pivot role resilience scores for ML Engineering occupations
4. Live tech stack fingerprint extracted from active job postings
5. Real-time project intelligence from public GitHub repos and ArXiv papers

It then checks whether those companies have live ML job postings via the ATS scanner, and enriches each shortlist entry with what the company is *actually building right now* — not just whether they can hire you.

**Use this mode when:**
- Your OPT end date is within 12 months and you need to prioritize applications ruthlessly
- You are targeting ML Engineering roles (SOC 15-1252, 15-2051, 15-1299) specifically, not generic SWE
- You want to tailor applications with real technical signal rather than generic cover letters

**Do not use this mode to:**
- Evaluate a single role in depth (use `modes/oferta.md` for that)
- Generate or scrape job postings (use `modes/scan.md`)

---

## Step 1 — Filter Companies by H-1B Sponsorship

**Data source (existing):**
`data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv`

**Run (existing script):**
```bash
python3 SCRIPTS/audit_sec_dol_h1b_data.py
```

**Manual filter (no unified script yet):**
```python
import pandas as pd

df = pd.read_csv("data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv")
h1b = df[df["h1b_employer"].notna() | df["total_lcas"].notna()]

# Target SOC codes for ML Engineering:
# 15-1252 Software Developers (most ML Eng LCA filings)
# 15-2051 Data Scientists
# 15-1299 Computer Occupations, All Other (MLOps, AI Platform)
target_socs = ["15-1252", "15-2051", "15-1299"]

# NOTE: SOC columns do not yet exist in this file (confirmed by audit).
# Cannot filter by SOC until SCRIPTS/lca/ is built. See Gap 1.
```

**⚠ Gap 1 — SOC columns missing from master file.**
Workaround: cross-reference by job title keyword using the BLS compact table in Step 3.

---

## Step 2 — Score by Funding Recency (SEC Form D)

**Data source (existing):**
```
data/sec/form-d/processed/companies-sec-2026q1-d.json
data/sec/form-d/processed/companies-sec-2025q4-d.json
data/sec/form-d/processed/companies-sec-2025q3-d.json
data/sec/form-d/processed/companies-sec-2025q2-d.json
```

**Run (existing script):**
```bash
python3 SCRIPTS/sec/refresh_recent_sec_quarters.py
```

**Manual merge (no recency-flag script yet):**
```python
import json, glob
import pandas as pd

frames = []
for f in glob.glob("data/sec/form-d/processed/companies-sec-202*.json"):
    with open(f) as fh:
        data = json.load(fh)
    quarter = f.split("companies-sec-")[1].replace("-d.json","")
    for co in data:
        co["sec_quarter"] = quarter
    frames.extend(data)

sec_df = pd.DataFrame(frames)
sec_df["recent_funding"] = sec_df["sec_quarter"].isin(["2025q4-d","2026q1-d"])
```

**⚠ Gap 2 — funding recency ≠ hiring signal.**
A 2026Q1 seed round of $2M is not the same as a Series B of $80M. Until a funding-stage/amount filter is added, treat `recent_funding = TRUE` as directional only. The GitHub intelligence step (Step 5) partially patches this — active repos signal active teams regardless of funding stage.

---

## Step 3 — Pull ML Role Resilience Scores from BLS Compact Table

**Data source (existing):**
`data/bls/compact/soc_occupation_compact.csv`

**Run (existing script):**
```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py
```

**What to look up:**

| SOC Code | Title | Why it matters for ML Eng |
|---|---|---|
| 15-1252 | Software Developers | Primary LCA filing code for ML Eng roles |
| 15-2051 | Data Scientists | ML research / applied science roles |
| 15-1299 | Computer Occupations NEC | MLOps, AI Platform, ML Infrastructure |

```python
bls = pd.read_csv("data/bls/compact/soc_occupation_compact.csv")
ml_socs = bls[bls["bls_soc_code"].isin(["15-1252","15-2051","15-1299"])][
    ["bls_soc_code","title","cognitive_pivot_score","job_zone","median_annual_wage"]
]
print(ml_socs)
```

Roles with `cognitive_pivot_score >= 3.5` and `job_zone >= 4` are the most resilient — they require system judgment and architecture work that AI cannot reliably replace.

**⚠ Gap 3 — cognitive_pivot_score is a first-pass score.** Use directionally, not as a precision measure.

---

## Step 4 — Check ATS and Job Liveness for Shortlisted Companies

**Data source (existing):**
`data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv` (column: `website`)

**Run (existing scripts):**
```bash
cd SCRIPTS/ats
python3 detect_ats.py \
  --csv ../../data/80-days-to-stay/data/SEC_DOL_H1b_data_mapped.csv \
  --company-column company_name \
  --output ../../data/ats/ml_shortlist_ats.csv
```

Configure `data/ats/portals.yml` with careers URLs for confirmed Greenhouse/Lever/Ashby companies, then scan:
```bash
npm run ats:scan
```

Check liveness for specific job URLs:
```bash
node SCRIPTS/ats/check-liveness.mjs https://boards.greenhouse.io/company/jobs/12345
```

**⚠ Gap 4 — ATS detection not yet run at full scale.**
Spot-check shortlisted companies manually until full-scale run completes.

---

## Step 5 — Tech Stack Fingerprint from Live Job Postings

**What this step does:**
When the ATS scanner pulls live ML job postings in Step 4, parse the requirements section to extract tech stack signals. This tells you whether to lead with PyTorch vs. JAX, mention Kubernetes vs. Ray, or highlight Triton/CUDA experience in your application.

**Proposed script (does not yet exist):**
`SCRIPTS/ml/extract_stack_signals.py`

**What it would do:**
```python
# Input: normalized job JSON from SCRIPTS/ats/scrapers/greenhouse/scraper.py
# For each job description, extract presence of:

STACK_SIGNALS = {
    "frameworks":   ["PyTorch", "JAX", "TensorFlow", "Flax"],
    "infra":        ["Kubernetes", "Ray", "Triton", "CUDA", "Slurm", "Airflow"],
    "cloud":        ["AWS SageMaker", "GCP Vertex", "Azure ML"],
    "focus":        ["RLHF", "fine-tuning", "inference optimization",
                     "distributed training", "RAG", "embeddings"],
}

# Output per company: stack_fingerprint.json
# {
#   "company": "Cohere",
#   "frameworks": ["JAX", "PyTorch"],
#   "infra": ["Kubernetes", "Ray"],
#   "focus": ["inference optimization", "fine-tuning"],
#   "source_jobs": ["Senior ML Engineer - Inference", "ML Platform Engineer"]
# }
```

**Output column to add to `ml_shortlist.csv`:** `stack_summary` (comma-separated top signals)

**What this verifies:** The tech stack claims are extracted directly from live job text — not inferred from the company's marketing page.

**What this does not verify:** Whether the stack listed in a job post reflects what the team actually uses day-to-day. Companies frequently list aspirational technologies.

---

## Step 6 — Real-Time Project Intelligence (GitHub + ArXiv)

**What this step does:**
For each company on the shortlist, check their public GitHub organization and recent ArXiv papers (filtered by company email domain in author affiliations) to see what ML problems they are *actually working on right now*. This is the most differentiated signal in the mode — it is not available from job posts, LinkedIn, or sponsorship data.

**Proposed script (does not yet exist):**
`SCRIPTS/ml/fetch_project_intelligence.py`

**GitHub org scrape (proposed logic):**
```python
import requests

def get_github_org_signals(org_name: str, token: str) -> dict:
    """
    Fetch public repos for a GitHub org, return:
    - repo names and descriptions
    - top languages
    - recently updated repos (pushed_at within 90 days)
    - topics/tags on repos
    """
    headers = {"Authorization": f"token {token}"}
    url = f"https://api.github.com/orgs/{org_name}/repos?sort=pushed&per_page=30"
    repos = requests.get(url, headers=headers).json()

    recent = [r for r in repos if r.get("pushed_at","") > "2026-03-01"]
    signals = {
        "active_repos": [r["name"] for r in recent],
        "descriptions": [r["description"] for r in recent if r["description"]],
        "languages": list({r["language"] for r in recent if r["language"]}),
        "topics": [t for r in recent for t in r.get("topics", [])],
    }
    return signals

# Example orgs to check:
# Cohere    -> "cohere-ai"
# Scale AI  -> "scaleapi"
# Mistral   -> "mistralai"
```

**ArXiv author affiliation scrape (proposed logic):**
```python
import arxiv

def get_arxiv_papers(company_domain: str, max_results: int = 10) -> list:
    """
    Search ArXiv for papers where author email matches company domain.
    Uses arxiv Python client (pip install arxiv).
    NOTE: ArXiv does not expose author emails directly in the API.
    Workaround: search by company name in affiliation field via semantic search
    or use Semantic Scholar API which indexes affiliations.
    """
    # Proposed: use Semantic Scholar API (free, no auth required for basic use)
    # https://api.semanticscholar.org/graph/v1/paper/search
    # Filter: affiliations contains company name, year >= 2024
    pass
```

**⚠ Gap 5 — GitHub org name ≠ company name in dataset.**
The master file has company names like "Cohere, Inc." but GitHub org names are informal ("cohere-ai"). A lookup table mapping normalized company names to GitHub org handles does not yet exist. Until it does, this step requires manual lookup for each shortlisted company.

**⚠ Gap 6 — ArXiv affiliation search is imprecise.**
The ArXiv API does not expose author email domains directly. The Semantic Scholar API is a better source but requires building a separate ingestion pipeline. Flag this as proposed, not implemented.

**Output to add to `ml_shortlist.csv`:** `github_org`, `active_repos_90d`, `arxiv_papers_2024`, `project_summary`

**Practical use:**
For each company that makes your final top-10 shortlist, run this step manually and add a `project_notes` field to your tracker. Example:

> *Cohere (2026-06-01): Active repos include `cohere-toolkit` (RAG framework, pushed 3 days ago) and `cohere-python` (SDK). Recent ArXiv: "Efficient Long-Context Attention via Sparse Kernels" (May 2026). Stack signal: inference optimization, JAX. Application angle: mention sparse attention work and JAX experience.*

---

## Step 7 — Assemble Final Shortlist and Score

**Proposed script (does not yet exist):**
`SCRIPTS/ml/build_ml_shortlist.py`

**Scoring formula (proposed, not automated):**

| Signal | Weight | Source | Verified? |
|---|---|---|---|
| H-1B sponsorship present | 30% | `SEC_DOL_H1b_data_mapped.csv` | ✅ |
| Recent Form D (2025Q4 or 2026Q1) | 25% | SEC Form D processed JSON | ✅ |
| Cognitive pivot score ≥ 3.5 | 20% | `soc_occupation_compact.csv` | ✅ first-pass |
| Stack match to student's skills | 15% | Live job post extraction | ⚠ proposed |
| Active GitHub repos in 90 days | 10% | GitHub API | ⚠ proposed |

---

## What This Mode Verifies vs. Infers

| Claim | Status | Source |
|---|---|---|
| Company has H-1B data in dataset | ✅ Verified | `SEC_DOL_H1b_data_mapped.csv` |
| Company filed Form D in last 2 quarters | ✅ Verified | `data/sec/form-d/processed/` |
| ML role cognitive resilience score | ✅ Verified (first-pass) | `soc_occupation_compact.csv` |
| Tech stack extracted from live job text | ✅ Verified (when script exists) | ATS job JSON |
| Company has active public GitHub repos | ✅ Verified (when script exists) | GitHub API |
| Company files H-1B for ML SOC codes specifically | ⚠ Inferred | No SOC filter yet |
| Job posting is currently live | ⚠ Inferred (until liveness check) | `check-liveness.mjs` |
| ArXiv papers reflect current team priorities | ⚠ Inferred | Semantic Scholar (proposed) |
| Company will sponsor your specific H-1B case | ❌ Not verifiable | Requires direct contact |

---

## Output Format

Save to: `data/ats/ml_shortlist.csv`

| company_name | h1b_present | total_lcas | recent_funding | sec_quarter | ats_provider | stack_summary | github_org | active_repos_90d | project_summary | composite_score |
|---|---|---|---|---|---|---|---|---|---|---|
| Cohere | TRUE | 40 | TRUE | 2026q1-d | Greenhouse | JAX, Ray, inference | cohere-ai | 4 | RAG toolkit, sparse attn | 0.91 |

---

## Log Template for modes/RUN_LOG.md

```
## ml-sponsorship-triage — [DATE]

**Inputs:**
- SEC_DOL_H1b_data_mapped.csv (30,369 rows)
- SEC Form D quarters: [list]
- soc_occupation_compact.csv (1,016 occupations)

**Steps completed:**
- [ ] audit_sec_dol_h1b_data.py — H-1B row count: ___
- [ ] SEC Form D recency flags added
- [ ] BLS SOC lookup for 15-1252, 15-2051, 15-1299
- [ ] ATS detection run for shortlist: Y/N
- [ ] Liveness checks run: Y/N
- [ ] Tech stack extraction run: Y/N (script exists: Y/N)
- [ ] GitHub org signals fetched: Y/N (script exists: Y/N)
- [ ] ArXiv/Semantic Scholar lookup: Y/N

**Output:** data/ats/ml_shortlist.csv — ___ companies
**Verified signals:** H-1B presence, Form D recency, cognitive pivot score
**Proposed signals run manually:** stack fingerprint, GitHub activity
**Gaps hit:** [list any]

**What to do next:**
- [ ] Build SCRIPTS/ml/build_ml_shortlist.py
- [ ] Build SCRIPTS/ml/extract_stack_signals.py
- [ ] Build SCRIPTS/ml/fetch_project_intelligence.py
- [ ] Run oferta.md on top 5 shortlist companies
```

# ux-designer-stem-opt.md
**Mode for The Reallocation Engine**  
Designed for: International MSIS / UX-adjacent students targeting UX/Product Design roles on OPT or future STEM OPT

---

## What this mode does and when to use it

This mode helps international students in MSIS, HCI, or UX-adjacent master's programs prioritize UX/Product Design job applications by checking verified signals before spending time on portfolio customization.

The core problem is not motivation; it is information asymmetry. A Product Designer or UX Designer posting may look promising on LinkedIn, but an international student often cannot easily see whether the posting is still live, whether the company has H-1B sponsorship history in design-adjacent SOC categories, whether the company has recent funding or stability signals, or whether HR is familiar with OPT/STEM OPT employer participation.

This mode uses verified data and tested scripts first. Prompting or LLM summarization may be used only after the data checks are complete, for explaining tradeoffs or drafting recruiter questions.

**Use this mode when:**
- You have a shortlist of target companies or job posting URLs.
- You are an international student applying for UX Designer, Product Designer, UX Researcher, Product Analyst, or PM-adjacent roles.
- You are on OPT, preparing for OPT, or planning for a future STEM OPT extension.
- You need to decide which applications deserve full portfolio customization versus quick-apply or deprioritization.

**Target user:** International master's student in Information Systems, HCI, Interaction Design, or a UX-adjacent STEM program, completing the program around August 2026 and targeting early-career Product Designer / UX Designer roles in the U.S.

**SOC codes in scope:**
- `27-1021` - Commercial and Industrial Designers
- `15-1299` - Web and Digital Interface Designers, or broader computer occupations depending on dataset labeling
- `15-1255` - Web and Digital Interface Designers, if present in the occupation table
- `11-2021` - Marketing Managers, used cautiously for PM-adjacent or growth/product roles
- `11-3021` - Computer and Information Systems Managers, used cautiously for PM-adjacent roles

**Important boundary:** SOC mapping for UX/Product Design is messy. This mode checks several plausible SOC categories, but it cannot prove how a specific employer would classify a specific future petition.

---

## Existing repo data or scripts used

This mode connects to the three engine layers described in the assignment.

### 1. 80 Days layer - sponsorship and funding signals

**Existing data layer used:** 80 Days to Stay company, funding, and H-1B sponsorship datasets.

**Data checks:**
- Company-level H-1B sponsorship history
- H-1B petition history for design-adjacent or product-adjacent SOC categories
- SEC Form D recency for private startups

**Proposed convenience commands:** The assignment repo contains the relevant data layer, but the following wrapper commands are proposed for this mode. They should not be treated as already implemented unless the repo later adds them.

```bash
# PROPOSED convenience command - not yet confirmed as implemented
python SCRIPTS/h1b_lookup.py --company "Figma" --soc 27-1021
python SCRIPTS/h1b_lookup.py --company "Figma" --soc 15-1299
python SCRIPTS/h1b_lookup.py --company "Figma" --soc 15-1255
python SCRIPTS/h1b_lookup.py --company "Figma" --soc 11-2021
python SCRIPTS/h1b_lookup.py --company "Figma" --soc 11-3021
```

```bash
# PROPOSED convenience command - not yet confirmed as implemented
python SCRIPTS/formd_lookup.py --company "Figma" --months 18
```

**Why these belong:** A UX/Product Design applicant needs a faster way to check whether a company has sponsored similar or adjacent roles before investing hours tailoring a portfolio case study.

---

### 2. Job-Ops layer - ATS and liveness checks

**Existing script path:**

```bash
node SCRIPTS/ats/check-liveness.mjs "https://example.com/job-posting-url"
```

**Data checked:**
- Whether the job URL is still live
- Whether the posting redirects or expires
- Whether the ATS provider is detectable, such as Greenhouse, Lever, or Ashby

**Why this matters for UX applicants:** UX/Product Design applications often require more customization than a resume-only application. A dead or stale role can waste 8-12 hours of portfolio tailoring.

---

### 3. Cognitive Pivot layer - role resilience and occupation fit

**Existing data layer used:** BLS / O*NET occupation table and SOC-based role-quality scoring layer.

**Existing or expected extraction command:**

```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py
```

**Proposed downstream scoring command:**

```bash
# PROPOSED downstream command - not yet confirmed as implemented
python SCRIPTS/bls/cognitive_score.py --soc 15-1255
```

**Why this belongs:** For an international student planning a multi-year OPT/STEM OPT/H-1B runway, the role should not only be available now; it should also remain valuable over the next 3-5 years. UX/Product roles with research, systems thinking, product judgment, stakeholder communication, and causal reasoning are stronger fits than roles that are mostly UI production or repetitive content generation.

---

## Proposed new data sources or commands

### A. `SCRIPTS/ux/ux_title_mapper.py` - proposed

```bash
python SCRIPTS/ux/ux_title_mapper.py --title "Product Designer, Growth" --job-description "jd.txt"
```

**Proposed output:**
- Likely SOC categories to check
- Role type: Product Designer / UX Designer / UX Researcher / Product Analyst / UI-heavy role / PM-adjacent role
- Ambiguity flag: low / medium / high

**Rationale:** UX titles are inconsistent across companies. A company may call the role Product Designer, UX Designer, Experience Designer, Digital Designer, or Product Analyst. Without a title mapper, the H-1B lookup may search the wrong SOC code and produce a false negative.

---

### B. `SCRIPTS/opt/stem_opt_hr_check.py` - proposed

```bash
python SCRIPTS/opt/stem_opt_hr_check.py --company "Figma" --degree-field "Information Systems"
```

**Proposed output:**
- Estimated STEM OPT familiarity: high / medium / low / unknown
- Evidence type: prior H-1B history, E-Verify public signal if available, manual recruiter confirmation required
- Confidence: low / medium / high

**Rationale:** STEM OPT requires employer participation, including Form I-983 and training-plan responsibilities. A company can have H-1B history but still be unfamiliar with STEM OPT employer requirements. This risk is especially important for international students in UX/Product Design because HR may not immediately understand how an MSIS or HCI degree connects to a design role.

**Honesty boundary:** This script does not currently exist in the mode. It is a proposed addition. Even if implemented, it would infer HR familiarity from indirect signals and would still require recruiter confirmation.

---

### C. `SCRIPTS/company/news_risk_check.py` - proposed

```bash
python SCRIPTS/company/news_risk_check.py --company "Figma" --days 90
```

**Proposed output:**
- Recent risk flags: hiring freeze, layoff, acquisition, funding shortfall, restructuring
- Source links or notes
- Risk level: low / medium / high / unknown

**Rationale:** H-1B and Form D data are retrospective. A company may have strong historical signals but a recent hiring freeze. This proposed check helps prevent the mode from over-trusting old sponsorship or funding history.

---

## Step-by-step workflow

### Step 1 - Build the input list

Create a list of 10-20 target jobs.

**Input fields:**
- Company name
- Role title
- Job posting URL
- Location
- Work model: onsite / hybrid / remote
- Source: LinkedIn, company career page, referral, recruiter, etc.
- Portfolio effort required: low / medium / high

---

### Step 2 - Check job liveness first [Job-Ops layer]

Run the existing liveness checker on each posting URL.

```bash
node SCRIPTS/ats/check-liveness.mjs "[posting URL]"
```

**Output:** live / expired / redirected / unable to verify; ATS provider if detectable.

**Decision rule:**
- Expired or redirected: do not customize portfolio unless a recruiter confirms the role.
- Live but older than 30-45 days: mark as potentially stale and manually verify.
- Live and recent: continue to sponsorship and fit checks.

---

### Step 3 - Map title to likely SOC categories [proposed UX layer]

Use the proposed title mapper or manual mapping.

```bash
# PROPOSED convenience command
python SCRIPTS/ux/ux_title_mapper.py --title "Product Designer, Growth" --job-description "jd.txt"
```

**Manual fallback:** Check several SOC codes instead of assuming one correct code.

**Decision rule:**
- If the role is UI-production-heavy and has low cognitive-demand fit, deprioritize unless it is a strong portfolio bridge.
- If the role includes research, product strategy, experimentation, systems thinking, or stakeholder decision-making, continue.

---

### Step 4 - Check H-1B history [80 Days layer]

Use the H-1B dataset or proposed wrapper command.

```bash
# PROPOSED convenience commands
python SCRIPTS/h1b_lookup.py --company "[Company]" --soc 27-1021
python SCRIPTS/h1b_lookup.py --company "[Company]" --soc 15-1255
python SCRIPTS/h1b_lookup.py --company "[Company]" --soc 15-1299
python SCRIPTS/h1b_lookup.py --company "[Company]" --soc 11-2021
python SCRIPTS/h1b_lookup.py --company "[Company]" --soc 11-3021
```

**Output:** Petition count, approval rate if available, median salary if available, year range.

**Decision rule:**
- 3+ relevant or adjacent petitions in the last 3 years: sponsorship signal is stronger.
- 1-2 petitions: sponsorship possible but weak; ask recruiter early.
- 0 petitions across all checked SOC codes: sponsorship-unverified; do not spend deep customization time unless there is a referral or strong reason.

---

### Step 5 - Check funding or company stability [80 Days layer]

For private startups, check recent Form D or other funding signals.

```bash
# PROPOSED convenience command
python SCRIPTS/formd_lookup.py --company "[Company]" --months 18
```

For public companies, skip Form D and log them as public-company stability check required. Do not use Form D as the main signal for large public companies.

**Decision rule:**
- Recent funding within 18 months: positive viability signal, but not proof of active hiring.
- No funding in 18 months for a small private company: funding-unverified.
- Public company: use company size, business unit stability, and live posting status instead.

---

### Step 6 - Check cognitive fit [Cognitive Pivot layer]

Use the BLS/O*NET occupation table and, if implemented later, the proposed cognitive scoring command.

```bash
python3 SCRIPTS/bls/extract_soc_occupation_table.py

# PROPOSED downstream command
python SCRIPTS/bls/cognitive_score.py --soc 15-1255
```

**Decision rule:**
- Strong fit: UX research, product judgment, systems design, cross-functional communication, causal reasoning.
- Medium fit: mixed visual/UI execution plus product thinking.
- Weak fit: repetitive production design, template editing, or purely visual asset generation with little product reasoning.

---

### Step 7 - Score priority

Assign one of four priorities.

| Priority | Meaning | Recommended action |
|---|---|---|
| Apply now | Strong liveness, sponsorship, and role-fit signals | Customize resume + portfolio case study |
| Research first | One major signal missing or weak | Ask recruiter / alumni / referral before deep work |
| Quick apply | Role may be useful, but signals are weak | Submit basic application only |
| Do not prioritize | Dead posting, no sponsorship signal, or poor role fit | Skip or revisit later |

---

## What this mode can and cannot verify

| Signal | Status | Source or method |
|---|---|---|
| Job posting live/dead status and ATS provider | Verified if liveness script succeeds | `SCRIPTS/ats/check-liveness.mjs` |
| Past H-1B petition count by company and SOC | Verified if dataset contains the company | 80 Days / H-1B dataset |
| Form D filing date and amount | Verified if company has filings | SEC Form D / 80 Days layer |
| Public-company status | Verified manually or through company dataset | Company lookup |
| SOC occupation table fields | Verified if extracted from BLS/O*NET table | Cognitive Pivot layer |
| UX title to SOC mapping | Inferred | Proposed `ux_title_mapper.py` or manual mapping |
| Role cognitive resilience | Partly inferred | BLS/O*NET fields + proposed scoring logic |
| STEM OPT HR familiarity | Inferred | Proposed `stem_opt_hr_check.py`; must confirm with recruiter |
| Whether the company will sponsor this candidate | Not verifiable | Requires recruiter / HR confirmation |
| Whether the specific team has budget | Not verifiable | Requires recruiter / hiring manager confirmation |
| Whether sponsorship policy changed recently | Not fully verifiable | Proposed news risk check may flag but cannot prove policy |
| Whether MSIS degree qualifies for STEM OPT in this case | Not verified by engine | Must verify through school/DSO and DHS STEM DDP list |

---

## Output format

| Company | Role | Location | ATS / liveness | H-1B signal | Funding / stability | Cognitive fit | STEM OPT HR familiarity | Priority | Notes |
|---|---|---|---|---|---|---|---|---|---|
| Figma | Product Designer, Growth | SF / hybrid | Live, Greenhouse | Example: strong historical signal | Public / mature private company; Form D not enough alone | Strong | Inferred high, recruiter confirmation needed | Apply now | Simulated example; verify current policy |
| StartupX | UX Designer | LA / hybrid | Live, Ashby | Unknown | No recent Form D found | Medium | Unknown | Research first | Ask recruiter before portfolio customization |
| AgencyY | UI Designer | Remote | Live, unknown ATS | No signal | Small company, funding unknown | Weak-medium | Unknown | Quick apply | Good visual practice, weak sponsorship signal |
| CompanyZ | Product Designer | Remote | Expired / redirected | Strong historical signal | Public company | Strong | Unknown | Do not prioritize current URL | Find a live posting or referral |

---

## Log template for `modes/RUN_LOG.md`

```md
## [DATE] - ux-designer-stem-opt run

**User situation:** International MSIS / UX-adjacent student, graduating around August 2026, targeting UX/Product Design roles in the U.S.

**Work authorization context:** OPT planned or active; future STEM OPT extension may be relevant. STEM eligibility must be confirmed with school/DSO and DHS STEM DDP list manually.

**Target roles:** Product Designer, UX Designer, UX Researcher, Product Analyst, PM-adjacent design roles

**Target locations:** [Los Angeles / Bay Area / New York / Remote / other]

**SOC codes checked:** 27-1021, 15-1255, 15-1299, 11-2021, 11-3021

**Companies checked:** [list]

**Existing scripts/data used:**
- `node SCRIPTS/ats/check-liveness.mjs`
- 80 Days H-1B sponsorship data
- 80 Days / SEC Form D funding data
- BLS/O*NET occupation table from Cognitive Pivot layer

**Proposed scripts used as simulation only:**
- `SCRIPTS/h1b_lookup.py` - proposed convenience wrapper
- `SCRIPTS/formd_lookup.py` - proposed convenience wrapper
- `SCRIPTS/ux/ux_title_mapper.py` - proposed
- `SCRIPTS/opt/stem_opt_hr_check.py` - proposed
- `SCRIPTS/company/news_risk_check.py` - proposed

**Verified findings:**
- Live postings found: [count]
- Expired / redirected postings found: [count]
- Companies with sponsorship history: [list]
- Companies with funding / stability concerns: [list]

**Inferred judgments:**
- STEM OPT familiarity estimates: [list]
- Role cognitive-fit estimates: [list]
- Application priority rankings: [list]

**Gaps / manual follow-up:**
- Recruiter questions needed: [list]
- SOC mapping uncertainty: [list]
- Current policy uncertainty: [list]

**Next action:**
- Apply now: [list]
- Research first: [list]
- Quick apply: [list]
- Do not prioritize: [list]
```

---

# Part 2 - Domain Justification

## Who uses this mode and in what situation

This mode is designed for an international master's student in Information Systems, HCI, or another UX-adjacent STEM program who is preparing for OPT or planning for a future STEM OPT extension. The student is targeting early-career UX Designer, Product Designer, UX Researcher, Product Analyst, or PM-adjacent roles.

This situation is specific because UX/Product Design applicants often invest significant time in each application. A strong application may require a tailored resume, reordered portfolio case studies, a custom case-study framing paragraph, and sometimes a short deck. Because the cost per application is high, a designer needs to know which jobs deserve deep effort.

## What information asymmetry this mode addresses

UX/Product Design students on OPT face three compounding blind spots.

**Blind spot 1 - Ghost postings waste portfolio effort.**  
Design roles often remain visible on LinkedIn or job boards after the company has stopped actively reviewing candidates. For a designer, this is expensive because a serious application can take hours of portfolio customization. The liveness check addresses this before the student invests effort.

**Blind spot 2 - Sponsorship history is hard to interpret for design roles.**  
Companies do not clearly publish whether they sponsor H-1B for UX or Product Design roles. Even when a company has H-1B history, the relevant petitions may be under inconsistent SOC categories. A Product Designer role might be classified under a design, digital interface, computer occupation, marketing, or product-adjacent code. This mode reduces false negatives by checking multiple plausible SOC categories.

**Blind spot 3 - STEM OPT HR process familiarity is invisible.**  
STEM OPT requires employer participation. HR may need to understand Form I-983, training-plan responsibilities, and E-Verify requirements. A company can be open to H-1B sponsorship later but still be unfamiliar with or unwilling to support STEM OPT. This risk is hard to see until late in the hiring process, so the mode flags it as a recruiter-confirmation question early.

## How it connects to the engine layers

- **80 Days layer:** Uses sponsorship history and funding signals to decide whether a company is worth deeper application effort.
- **Job-Ops layer:** Uses ATS detection and job liveness checking to avoid wasting portfolio customization time on dead or stale postings.
- **Cognitive Pivot layer:** Uses BLS/O*NET occupation data to prioritize roles that rely on product judgment, research interpretation, systems thinking, and causal reasoning instead of purely repetitive UI production.

## Failure modes

**Failure mode 1 - SOC code mismatch creates false negatives.**  
A Product Designer or UX Designer role may be filed under a different SOC code than expected. If the mode checks only one SOC category, it may label a company as sponsorship-unverified even when the company has sponsored similar roles under another code. This is hardest to catch because the output looks clean: zero petitions appears objective, but the search may have used the wrong label.

**Failure mode 2 - STEM OPT familiarity is indirect and lagged.**  
The proposed `stem_opt_hr_check.py` cannot directly confirm whether HR currently processes Form I-983 or supports STEM OPT extensions. It would infer familiarity from indirect evidence such as past sponsorship timelines or public employer signals. A company that changed policy after layoffs, restructuring, or HR leadership changes might still appear safe based on old data.

**Failure mode 3 - Historical data misses current hiring intent.**  
H-1B and funding records are retrospective. A company with strong historical sponsorship and funding signals may currently have a hiring freeze, team-specific budget cut, or paused sponsorship policy. This is why the output should trigger recruiter questions rather than be treated as a final decision.

---

# Part 3 - Worked Example: Simulated Run

## Scenario

**Student:** International MSIS student graduating around August 2026, targeting UX/Product Design roles in the U.S.  
**Target role:** Product Designer, Growth  
**Target company:** Figma  
**Posting source:** Company career site / Greenhouse-style job URL  
**Work authorization context:** OPT planned; STEM OPT may be relevant later if degree eligibility is confirmed by school/DSO.

**Important note:** This is a simulated run for demonstration. The values below show the expected output format and decision logic. They should not be read as audited current Figma data unless the underlying repo datasets and scripts are actually run.

## Inputs used

- Company: Figma
- Role title: Product Designer, Growth
- Job URL: `[example Greenhouse URL]`
- SOC codes checked: 27-1021, 15-1255, 15-1299, 11-2021, 11-3021
- Student target: UX/Product Design roles requiring portfolio customization

## Simulated mode output

```text
1) Liveness check
Command: node SCRIPTS/ats/check-liveness.mjs [example Greenhouse URL]
Output: LIVE; ATS provider detected: Greenhouse; posting appears recent
Status: verified if script succeeds

2) H-1B sponsorship check
Command: proposed SCRIPTS/h1b_lookup.py across five SOC codes
Output: historical sponsorship signal found in at least one design-adjacent or product-adjacent SOC category
Status: simulated; would be verified only after running against the dataset

3) Funding / stability check
Command: proposed SCRIPTS/formd_lookup.py --company "Figma" --months 18
Output: company stability requires manual interpretation because mature/private or public-company context may make Form D insufficient alone
Status: partially verified if filings are found; interpretation remains manual

4) Cognitive fit check
Command: BLS/O*NET occupation table + proposed cognitive scoring logic
Output: Product Designer, Growth likely has strong cognitive fit if the role includes experimentation, product judgment, research interpretation, and cross-functional decision-making
Status: inferred from role description and occupation table, not fully verified

5) STEM OPT HR familiarity check
Command: proposed SCRIPTS/opt/stem_opt_hr_check.py
Output: NOT IMPLEMENTED. Manual recruiter confirmation required.
Status: not verified
```

## Priority table

| Company | Role | ATS / liveness | H-1B signal | Stability signal | Cognitive fit | STEM OPT familiarity | Priority |
|---|---|---|---|---|---|---|---|
| Figma | Product Designer, Growth | Live, ATS detected | Historical signal to be checked across multiple SOCs | Needs current context check | Strong if role includes product judgment and experimentation | Not verified; ask recruiter | Apply now, but confirm OPT/STEM OPT policy before deep customization |

## Verified vs. inferred

| Data point | Status | Explanation |
|---|---|---|
| Posting live/dead status | Verified if `check-liveness.mjs` succeeds | The script can check whether the URL is still active |
| ATS provider | Verified if detected | Greenhouse/Lever/Ashby detection can be logged |
| H-1B history | Verified only after dataset lookup | The mode checks multiple SOC codes to reduce false negatives |
| Funding / stability | Partly verified, partly interpreted | Form D can verify filings, but not current hiring intent |
| Cognitive fit | Inferred | Based on SOC table plus job-description interpretation |
| STEM OPT HR familiarity | Not verified | Proposed script does not exist; recruiter confirmation required |
| Current sponsorship policy | Not verifiable by this mode | Must ask recruiter or HR |

## What I would change after seeing the output

The worked example shows that the mode is useful, but the most important missing signal is not another LLM summary. It is a recruiter-facing verification step for OPT/STEM OPT support.

I would add two follow-up outputs:

1. A generated recruiter question:
   > I am currently planning my U.S. work authorization timeline. Before moving forward, could you confirm whether the company supports OPT and, if applicable later, STEM OPT employer participation such as Form I-983?

2. A current-company-risk check:
   - Search recent company news for hiring freezes, layoffs, acquisition events, or sponsorship policy changes.
   - Log this as a risk flag, not as a verified decision.

**Main learning:** STEM OPT changes the shape of the risk, not the size. A longer work-authorization runway reduces immediate time pressure, but it introduces a new blind spot: whether HR understands and supports the employer-side process.

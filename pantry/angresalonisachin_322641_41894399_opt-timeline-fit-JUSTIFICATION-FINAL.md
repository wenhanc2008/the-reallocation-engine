# Domain Justification: opt-timeline-fit Mode

## Who Uses This Mode

International students on F-1 OPT visa (6–12 months before expiration) seeking H-1B sponsorship. Specifically: those applying to tech/finance/product roles in US hubs (Bay Area, NYC, Seattle) who need to know which companies sponsor visas AND whether hiring timelines fit their OPT deadline.

## Information Asymmetry Addressed

Students don't see:
- **Which companies sponsor H-1B:** 30,369 funded US companies exist; only 1,557 (5.1%) have H-1B sponsorship records. Job boards don't filter by sponsorship likelihood.
- **Hiring timelines vs. OPT window:** A company may sponsor H-1B but take 6+ months to hire. If student has 6 months of OPT remaining, that company doesn't work—but this is invisible upfront.
- **Role quality signals:** Not all product manager roles have equal visa sponsorship strength. Some signal rare skills (visa stronger); others don't (visa weaker).

Result: Students apply blindly to 200 companies; 3 actually sponsor H-1B for their role; 1 has a hiring timeline that fits their OPT window.

## Connection to Engine Layers

**Layer 1 (80 Days to Stay):** Does this company sponsor H-1B?
- Data: SEC Form D funding + DOL LCA + USCIS approval rates
- Output: Sponsorship tier (Proven ≥50 approvals / Likely 10-50 / Established 1-10 / None)

**Layer 2 (Job-Ops):** Is the job posting real and active?
- Data: ATS detection (Greenhouse/Lever) + HTTP liveness check
- Output: Live / Stale / Dead (checked via Playwright)

**Layer 3 (Cognitive Pivot):** Does this role's skill profile strengthen visa case?
- Data: BLS/O*NET SOC code + cognitive_pivot_score (reasoning, judgment, systems thinking)
- Output: Narrative strength (Strong >4.0 / Moderate 3.5-4.0 / Weak <3.5)

Mode ranks companies = sponsorship_tier × liveness × role_fit × timeline_factor

## Failure Modes: What Gets Wrong, For Whom

**1. Historical Sponsorship ≠ Current Policy**
- Company sponsored 15 H-1Bs three years ago; student assumes it still does. Company froze sponsorship due to immigration policy change or cost-cutting.
- Hardest for: Students who assume historical data = current policy; who don't verify with current employees before investing time.

**2. Visa Category Mismatch (Cap-Subject vs. Cap-Exempt)**
- Mode assumes cap-subject H-1B (most common). Student doesn't know they're cap-exempt (prior H-1B or US master's degree). Cap-subject filing window closes April 30; student graduates August 12. Pathway won't work.
- Hardest for: Students who don't verify cap-exempt eligibility with immigration lawyer before job search; who assume "H-1B" is one pathway.

**3. Hiring Timeline Too Slow for OPT Window**
- Company has good H-1B history. Hiring takes 4+ months. Student has 6 months of OPT. Offer comes in month 5; H-1B approval comes in month 10 (after OPT expires).
- Hardest for: Students who optimize for fresh postings without asking actual hiring timeline; who trust generic "H-1B takes 3 months" estimates.

**4. Posting Freshness ≠ Hiring Speed**
- Job posted 2 days ago (signals company is hiring). But company has slow approval chain (multiple rounds, VP review). Hiring closes in 12 weeks. Student is near OPT expiration by offer time.
- Hardest for: Students who treat posting age as hiring speed proxy; who don't gather insider timing data early.

**5. Cap-Exempt Inferred Incorrectly from H-1B History**
- Student sees company has 100 H-1B employees; infers company sponsors cap-exempt. Wrong logic. Cap-exempt is employee attribute (prior H-1B or US master's), not company attribute.
- Hardest for: Students who self-assess visa status without lawyer; who conflate company H-1B volume with personal eligibility.

## Why This Matters

International students represent ~15% of tech/product workforce in major US hubs but face an invisible constraint: visa sponsorship timelines. This mode makes visible what job boards hide: which 1,557 of 30,369 companies sponsor H-1B, which postings are actually live, and which timelines fit the student's OPT deadline. For students with 6 months of work authorization and only one legal pathway (H-1B sponsorship) to stay in the US, these signals are the difference between landing a visa and aging out.

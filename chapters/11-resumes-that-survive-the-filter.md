# Chapter 11 — Resumes That Survive the Filter
*The first reader doesn't care how it looks.*

There is a particular kind of effort that feels productive but costs you exactly what you spent it on. A candidate spends a weekend on a résumé. Two columns, a sidebar with tasteful skill bars, a small icon next to each section header, the name set large in a header graphic. It is the best-looking document they have ever made. They submit it to forty companies. Most reject it before a human opens it — not because the content was weak, but because the applicant-tracking system that read it first saw a scrambled wall of text. The two columns interleaved into nonsense. The skill bars, rendered as graphics, produced no text at all. The name, trapped in an image, was simply absent. The parser could not find a job title. So it scored the candidate as unqualified for everything, and the beautiful résumé disappeared.

This is not a corner case. By 2025, roughly 82% of companies screened résumés with software, and about one in five candidates were auto-rejected with no human review.[^screening] The first reader of your résumé is a parser. The parser does not care that the document is beautiful. It cares whether it can extract structured text. A résumé a human would admire and a machine cannot read is a résumé no human will ever see.

<!-- → [DIAGRAM: two parallel paths from "Submit résumé" — left path labeled "ATS-parseable": document → parser extracts name/title/dates cleanly → human review → interview; right path labeled "Designed layout": document → parser sees scrambled text or nothing → auto-reject → no human ever opens it; the fork happens before any human judgment] -->

I want to be precise about what this means, because the instinct is to fight it. "A distinctive design will make me stand out." Against a human, sometimes — if the human ever gets there. Against the parser that reads first, a distinctive design is how you disappear. Standing out is the job of your content and your portfolio. The résumé's job is narrower and absolute: pass the parser, then say something true and specific to the human behind it. Those are two different tasks, and conflating them is what produces the beautiful invisible résumé.

---

The pipeline this chapter rests on is `SCRIPTS/resumes/`, whose core is `generate-pdf.mjs`: a script that takes a Markdown CV and renders it to a PDF through Playwright/Chromium using a resume-safe rendering path — single-column, real text, standard section headings, no layout the parser will trip on. The whole point is that you author content in plain Markdown and let the pipeline produce a document engineered to parse. You do not hand-build a layout that looks good and reads as garbage. You write text, and the pipeline handles the document.

From the project root:

```bash
npm run resumes:pdf      # runs SCRIPTS/resumes/generate-pdf.mjs
```

The output is a rendered PDF. But the real check is not "does it look right." It is "does it parse." So there is a second, essential step: copy all text from the PDF — Ctrl/Cmd-A, Ctrl/Cmd-C — and paste it into a plain text editor. What you see pasted is roughly what the parser sees. If your name, titles, and dates come through as clean linear text in the right order, the document passes. If they scramble or vanish, you have found the break before a company did.

<!-- → [TABLE: two-column table showing "What you designed" vs. "What the parser extracted" for five common layout choices — two-column layout, skill bars as graphics, name in header image, dates in a table cell, section icons — right column shows the parser's view: interleaved text, blank, absent, misaligned, nothing] -->

That paste test is a practitioner heuristic, not a certificate. ATS systems vary. What one parses cleanly another may struggle with. But if text scrambles or disappears in plain paste, it will scramble or disappear in some meaningful fraction of the systems reading your application. The heuristic is conservative in the right direction: it tells you where the floor is, and the floor is what you need to be above.

---

Let me walk through what the test actually reveals, using a single CV rendered two ways.

The Markdown version, run through `resumes:pdf`: the PDF looks clean. Run the paste test. Name, contact information, each role's title and dates, the bullet content — all come through as ordered linear text. Section headings ("Experience," "Education," "Skills") survive as plain words, which are exactly the words parsers key on. The test passes.

The same content in a two-column designed template: paste it out. The left and right columns interleave line by line. "Data Analyst, 2023–2024" lands in the middle of an unrelated bullet from the other column. The skills section, rendered as little bars with labels, produces nothing — no text extracted at all. The parser reading that document cannot reliably locate a single job title. It has a name problem, a date problem, and a skill problem, all at once, and the human who might have found the candidate compelling never gets the chance to try.

<!-- → [CHART: side-by-side paste-test output for the same CV rendered two ways — left: clean linear text with name, title, dates, bullets in order; right: scrambled interleaving with missing sections marked in red; caption should note that the right-side output is what a typical two-column designed template actually produces when parsed] -->

The decision that follows from this is simple to state and genuinely uncomfortable to execute: ship the single-column rendered version; keep the pretty one for nothing, or for a human-only context like a portfolio site. The discomfort is real. The designed version *is* better-looking. But it is better-looking for a reader it will never reach.

---

Now let me say what the structures are that break parsers, because "don't use a fancy layout" is advice without traction until you know exactly which choices cause the problem.

**Multi-column layouts** are the most common failure mode. A parser reads text in document order, which in a two-column PDF is typically left-to-right across the full page width, not column by column. So the left column's first line and the right column's first line appear interleaved. This is not a subtle degradation — it makes the extracted text incoherent.

**Text in images** is the second. A name set in a header graphic is invisible to the parser. Literally absent. The parser cannot do OCR; it reads embedded text. If your name is an image, the parser does not know your name. If your section headers are icons rather than text characters, the parser does not see section structure.

**Tables used for layout** create extraction problems similar to columns: the parser linearizes the cells in ways that depend on the table's internal structure, not the visual arrangement you intended. Dates and job titles drift into wrong positions. A table is appropriate for genuinely tabular data; it is destructive when used to control visual layout.

**Skill bars and graphic elements** produce no text at all. They exist visually. To the parser, they are a gap.

The safe structures are their opposites: a single-column flow, real text characters for every piece of content that needs to be extracted, standard headings as plain words, dates in a consistent parseable format next to the roles they describe. These are not design constraints that prevent a good résumé. They are the constraints that ensure the résumé is read.

<!-- → [INFOGRAPHIC: a labeled résumé showing "safe zones" and "danger zones" — safe: single-column body text, standard section headers as text, dates as plain text adjacent to roles; danger: two-column layout, header graphic, skill bars, table-based layout, any text in an image; designed as a quick-reference visual a student can check their own document against] -->

---

There is something the pipeline cannot do, and I want to be direct about it.

The pipeline can guarantee a parseable document. It cannot know which of your true accomplishments will land with this hiring manager. It cannot tell you that your most parser-friendly bullet is also your most forgettable one. It cannot identify the phrase in your summary that makes a reader lean forward. ATS-safety is a floor — it ensures the human gets to read you at all. What you say once you are through the gate, and whether it is specific and true enough to earn the interview, is judgment the parser was never measuring.

The résumé, in a market where roughly 82% of companies run applications through software first, is not where you win. It is a gate you must not lose at. The winning moves — portfolio, proof of capability, demonstrated skill — happen elsewhere. The résumé's job is to get through the first filter intact and give the human on the other side something true and specific to respond to. Those are two requirements, and they are both necessary. An ATS-safe PDF with weak content still fails the human.

<!-- → [DIAGRAM: a two-stage model showing the résumé's job — Stage 1: "ATS filter" with the criterion "Can the parser find your name, title, and dates?" — Stage 2: "Human review" with the criterion "Is the content specific and true enough to earn a call?" — with an arrow between them labeled "What survives Stage 1 is the floor; Stage 2 is where content quality decides"] -->

Every component in the engine now works on its own: funding, sponsorship, liveness, role quality, timeline, scoring, framing, and a résumé that survives the filter. The next act stops handing you clean single tasks. It runs the whole engine — under real pressure, with a log — and that changes what you are doing in a way a single chapter cannot fully prepare you for.

---

## Exercises

**Warm-up**

1. *(Recall — single-concept)* Explain why a two-column PDF layout causes parser failures even when all the text is real text (not images). What is the parser actually doing when it reads a two-column document, and what does that produce?
   *Tests: understanding linearization as the mechanism, not just "columns are bad."*
   *Difficulty: Low*

2. *(Identify — single-concept)* Name four document structures that cause parsers to fail or produce scrambled output. For each, state what the parser sees instead of the intended content.
   *Tests: knowledge of the specific failure modes, not just the general principle.*
   *Difficulty: Low*

3. *(Apply — procedure)* Describe the paste-into-plain-text test step by step. What does it check? What does a passing result look like, and what does a failing result look like?
   *Tests: ability to execute and interpret the primary verification method.*
   *Difficulty: Low*

**Application**

4. *(Analyze — document)* A classmate sends you their résumé to review before they run `npm run resumes:pdf`. The document uses a two-column layout, has the candidate's name in a large header graphic, and uses icon symbols (not text) for section headers like Experience and Education. Before they render it, list every specific problem the paste test will likely reveal and explain the fix for each.
   *Tests: diagnosing multiple failure modes in a single document; pairing diagnosis with correction.*
   *Difficulty: Medium*

5. *(Evaluate — tradeoff)* A candidate argues they should keep a beautifully designed PDF for applications to design or creative roles because "those hiring managers will appreciate the aesthetic." Evaluate this argument. Under what specific conditions, if any, is it correct? What would need to be true about the application process for the designed version to be the right choice?
   *Tests: identifying the condition under which the design argument is valid — human-first review — vs. the much more common ATS-first case.*
   *Difficulty: Medium*

6. *(Apply — fix)* You have a CV section that lists technical skills as a visual grid: six cells arranged in two rows of three, each cell containing a skill name and a small logo. Rewrite this section in a form that passes the paste test without losing any of the skill information. Show the before structure and the after structure.
   *Tests: translating a visual layout element into parseable text without content loss.*
   *Difficulty: Medium*

**Synthesis**

7. *(Integrate — cross-chapter)* A student has a Proven-tier target company, a high timeline factor, and strong fit — all the signals from Chapters 5 through 10 point to applying. They run `npm run resumes:pdf`, do the paste test, and find their name is absent (it was in a header graphic) and two job titles scrambled. Walk through the decision: apply now with the broken résumé, delay to fix it, or something else? Include the timeline reasoning from Chapter 8 in your answer.
   *Tests: integrating ATS-safety as a go/no-go condition with the timeline urgency of Chapter 8.*
   *Difficulty: High*

8. *(Diagnose — system limits)* A student runs the paste test, sees clean linear text, and concludes their résumé will pass every ATS. Identify the specific conditions under which this conclusion is wrong — where the paste test gives a false pass — and explain what additional checks, if any, are available.
   *Tests: understanding the paste test as a strong heuristic with known limits, not a certificate.*
   *Difficulty: High*

**Challenge**

9. *(Design — open-ended)* The current pipeline converts Markdown to a single-column ATS-safe PDF. Propose a verification layer that would give a student more confidence than the paste test alone — for example, something that checks for specific parser-friendly conditions programmatically rather than by visual inspection. Describe what it would check, what output it would produce, and what it still could not guarantee.
   *Tests: thinking about what "ATS-safe" means as a verifiable property, not just a style guideline; understanding what any local check cannot know about a remote ATS.*
   *Difficulty: High/Open-ended*

[^screening]: ~82% of companies screen résumés with AI; ~21% auto-reject without human review — "The Collapse of the Traditional Résumé" (N. Bear Brown). **[verify]** against primary source.

[^proof]: The résumé-as-gate framing and the ~56% validated-AI-skill wage premium are from "The Collapse of the Traditional Résumé." **[verify]** the wage-premium figure before publication.

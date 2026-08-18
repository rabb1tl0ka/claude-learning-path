---
name: exam-analysis
description: Reads a mock-exam result PDF/text (dropped in 2ndbrain/exam-prep/attempts/) and reports failure and strength patterns in plain language — not just which domain scored low, but why: recurring reasoning mistakes (e.g. picking an infra fix for a reasoning problem, undervaluing role-scoping) and what you consistently get right. Optionally takes a second path to a Gemini transcript of Bruno taking the exam out loud, to enrich the "why" with his real-time reasoning. Saves a dated analysis note, then recommends running /flashcards import on the same file. Triggers on "/exam-analysis", "analyze my exam results", "process this exam PDF", "why am I weak at X".
argument-hint: <path-to-exam-result-file> [path-or-link-to-reasoning-transcript]
---

# /exam-analysis — Failure-pattern report from a mock exam

Runs against a single exam-result file (PDF or text) that Bruno has already saved to `2ndbrain/exam-prep/attempts/<source-slug>-<date>.<ext>`. This is a read-and-report step — it does not touch `.flashcards/` at all (that's `/flashcards import`'s job, which this skill recommends as its last step).

An optional second argument points to a transcript (e.g. a Gemini transcript of a self-recorded session where Bruno read each question and thought out loud in real time while taking the exam). There's no naming convention or auto-discovery for this file — only use it when Bruno explicitly passes a second path or link. This can be a Google Doc link — read it directly via the Drive tools (or fetch it however fits the link) and use it from context. Don't write a local copy of it to disk; it's a read-once input to this run, not an artifact worth persisting, and Bruno already has the live original.

## Step 1: Read the file(s)

Read the full exam-result file (PDF: all pages, batching per the Read tool's page-range limit if long). Expect a domain/category breakdown (score per domain) and a question-by-question review: question text, the answer given, the correct answer, an explanation, and (if present) a "where this comes from" lesson reference. Formats vary between exam sites — be liberal about what counts as a question record, same as `/flashcards import`'s Step 2b.

If a transcript was given, read it too (from disk if it's a local path, straight from source if it's a link — see above). It won't be structured like the exam file — it's Bruno narrating each question as he reads and answers it live. Match transcript segments to exam questions by content (question text/topic), not by assuming a fixed order or one-to-one segment boundaries.

## Step 2: Extract the raw signal

For every question, note: domain/category, correct vs incorrect, the option picked, the correct option, and the explanation text. Don't skip correct answers — strengths need the same evidence as weaknesses.

When a transcript is available, also note, per question, what it reveals about Bruno's real-time reasoning — e.g. he considered the correct answer and talked himself out of it, ran low on time and guessed, misread the question, or was confidently wrong throughout. This is supplementary evidence: the exam file's question/answer/explanation stays the backbone of the analysis, and the transcript enriches or sharpens the "why" — it doesn't override the explanation's account of what the correct reasoning was.

## Step 2b: Check every question against course notes and gap-topics notes

Separately from Steps 2-3 (which analyze failure/strength patterns across the whole exam), run two independent checks per question — not a mutually-exclusive 3-way classification, just two yes/no lookups:

- **Course-mapped?** Does the question's actual tested concept get taught by an existing chapter note under one of the 4 completed courses (`2ndbrain/ccaf-learning/claude-code-agent-skills/`, `claude-api/`, `model-context-protocol/`, `claude-code-in-action/`)? Judge this the same way `/flashcards import`'s Step 2 does — by subject matter, not by whether the exam's own domain label or "where this comes from" citation (if present) sounds similar; a third-party exam site's own lesson numbering is not evidence of official-course coverage. If yes, record which chapter note.
- **Gap-mapped?** Does it match an existing note under `2ndbrain/ccaf-learning/**/*.md` that isn't one of the 4 course dirs (a promoted gap-topic dir, e.g. `ccaf-learning/structured-data-extraction/`)? If yes, record which note. (In practice this won't overlap with course-mapped — gap-topic notes exist specifically for material the courses don't cover.)

Questions that are neither don't need a label or a bucket of their own — they're just not part of either report below, and discovering whether they deserve a *new* gap-topics note is `/flashcards import`'s job (its Step 3), not this skill's. Don't create new gap-topics notes here, and don't force a course match just to give every question a home.

The actual certification exam is the CCAF (Claude Certified Architect – Foundations), scoped to Claude Code, the Claude Agent SDK, the Claude API, and MCP broadly — not just the 4 official Claude Learning Path courses (per `exam-prep/CLAUDE.md`). So the **raw overall score is the number closest to real CCAF readiness** — don't compute or surface a separate "course-mapped accuracy" as if it were the truer readiness signal; course-mapped vs gap-mapped is a routing distinction (which notes to update), not a scoping one.

Still compute the course-mapped and gap-mapped breakdowns from Step 2b, but report them as **informational routing info only** — where a wrong answer's underlying concept lives (an existing chapter note vs an existing gap-topics note vs neither) — not as competing readiness metrics.

## Step 3: Find patterns, not just percentages

A domain breakdown table only says *where*. The point of this skill is *why*. Group the incorrect answers by the kind of mistake, using the explanation text as evidence — for example: picking a fix that adds infrastructure/tooling when the real problem is a reasoning/prompt/architecture gap (or vice versa); treating a hardcoded rule as adequate when the question calls for model-driven judgment; missing scope/isolation issues (shared config vs personal, stale context, role bleed between agents); confusing two similar built-in tools/mechanisms (picking the technically-workable option over the recommended/idiomatic one); or any other pattern that recurs 2+ times. Name each pattern plainly and quote the 1-2 clearest example questions as evidence.

Do the same grouping for correct answers, but lighter-touch: 1-2 sentences on what domains/patterns show genuine strength, so the report isn't purely deficit-framed.

Do not force a pattern that isn't there — a single one-off wrong answer with no sibling doesn't need its own category. Group those under a short "other misses" note.

When a transcript is available, also look for patterns in *how* Bruno arrives at wrong answers that the exam explanations alone can't show — e.g. consistently second-guessing a correct first instinct, running short on time on certain domains, misreading questions under time pressure. Only name one of these as its own pattern if it recurs 2+ times, same bar as any other pattern; otherwise fold the observation into the relevant failure pattern's evidence.

## Step 4: Report in chat

Give Bruno, in plain conversational text (no giant tables — he already saw the domain breakdown in the PDF itself): the overall score/pass-fail in one line (this is the number closest to real CCAF readiness, since the exam covers this material broadly, not just the 4 official courses); 2-4 named failure patterns, each with what it is, why it happens (per the explanations, sharpened by the transcript's real-time reasoning when available), and which questions demonstrate it; 1-2 named strength patterns; and any questions that didn't fit a pattern (the "other misses" bucket), briefly. If Step 2b found any gap-topics matches, mention them briefly too (a note may be worth a quick read/update if this attempt revealed something new about it).

## Step 5: Save the analysis note

Write the same report to `2ndbrain/exam-prep/attempts/<source-slug>-<date>-analysis.md` (same slug/date as the source file, `-analysis` suffix) so patterns are comparable across multiple exam attempts over time. Use the source file's own name to derive `<source-slug>-<date>` if it already follows that convention; otherwise ask Bruno for a slug.

Include a `## Course-topic performance` section as a markdown table, grouped by chapter note (not one row per question — same style as `## Domain breakdown`): columns `Chapter note` (linked), `Score` (`X/Y (Z%)`, with `— weakest`/`— strongest` on the extremes same as the domain table), `Wrong` (the missed question numbers, or `—` if none).

Include a `## Gap-topic performance` section in the same grouped style (only if Step 2b found any matches): `Gap-topic note` (linked), `Score`, `Wrong`. Note in a line above the table that this is informational routing (which existing gap-topics note this maps to), not a separate readiness metric — these questions count toward the overall score like any other, since gap topics are real CCAF exam scope.

## Step 6: Recommend the next step

Point Bruno to `/flashcards import <path-to-original-exam-file>` as the next action — don't run it automatically, since this skill's job ends at analysis.

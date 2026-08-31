---
name: exam-analysis
description: Reads a mock-exam result PDF/text (dropped in 2ndbrain/exam-prep/attempts/) and reports failure and strength patterns in plain language — not just which domain scored low, but why: recurring reasoning mistakes (e.g. picking an infra fix for a reasoning problem, undervaluing role-scoping) and what you consistently get right. Also writes a per-question coaching explanation for every question (right or wrong) into the saved note, same teaching-not-just-verdicting spirit as /flashcards' quiz coaching. Optionally takes a second path to a Gemini transcript of Bruno taking the exam out loud, to enrich the "why" with his real-time reasoning. Drafts or updates cross-cutting reasoning heuristics in the exam-prep survival guide when a failure pattern is a general disambiguation rule rather than a single-topic gap. Saves a dated analysis note, then recommends running /flashcards import on the same file. Triggers on "/exam-analysis", "analyze my exam results", "process this exam PDF", "why am I weak at X".
argument-hint: <path-to-exam-result-file> [path-or-link-to-reasoning-transcript]
---

# /exam-analysis — Failure-pattern report from a mock exam

Runs against a single exam-result file (PDF or text) that Bruno has already saved to `2ndbrain/exam-prep/attempts/<source-slug>-<date>.<ext>`. This is a read-and-report step — it does not touch `.flashcards/` at all (that's `/flashcards import`'s job, which this skill recommends as its last step).

An optional second argument points to a transcript (e.g. a Gemini transcript of a self-recorded session where Bruno read each question and thought out loud in real time while taking the exam). There's no naming convention or auto-discovery for this file — only use it when Bruno explicitly passes a second path or link. This can be a Google Doc link — read it directly via the Drive tools (or fetch it however fits the link) and use it from context. Don't write a local copy of it to disk; it's a read-once input to this run, not an artifact worth persisting, and Bruno already has the live original.

## Step 1: Read the file(s)

Read the full exam-result file (PDF: all pages, batching per the Read tool's page-range limit if long). Expect a domain/category breakdown (score per domain) and a question-by-question review: question text, the answer given, the correct answer, an explanation, and (if present) a "where this comes from" lesson reference. Formats vary between exam sites — be liberal about what counts as a question record, same as `/flashcards import`'s Step 2b.

If a transcript was given, read it too (from disk if it's a local path, straight from source if it's a link — see above). It won't be structured like the exam file — it's Bruno narrating each question as he reads and answers it live. Match transcript segments to exam questions by content (question text/topic), not by assuming a fixed order or one-to-one segment boundaries.

Gemini meeting transcripts/notes often carry their own **"Next steps"** section separate from the raw transcript body, listing action items tagged with an owner (e.g. `[Claude]`, `[Bruno Coelho]`, `[GL]`, `[The group]`). Check for one. Any item tagged for Claude (or otherwise clearly addressed to Claude/"you") is a task to actually do as part of this run, not just background color for the reasoning analysis — resolve it (investigate, look something up, compute an answer) and report the result explicitly in Step 4 and record it in Step 5's saved note, even when it isn't itself a failure-pattern or heuristic. Items tagged for Bruno or a third party are just context, not something to execute.

## Step 2: Extract the raw signal

For every question, note: domain/category, correct vs incorrect, the option picked, the correct option, and the explanation text. Don't skip correct answers — strengths need the same evidence as weaknesses.

When a transcript is available, also note, per question, what it reveals about Bruno's real-time reasoning — e.g. he considered the correct answer and talked himself out of it, ran low on time and guessed, misread the question, or was confidently wrong throughout. This is supplementary evidence: the exam file's question/answer/explanation stays the backbone of the analysis, and the transcript enriches or sharpens the "why" — it doesn't override the explanation's account of what the correct reasoning was.

## Step 2a: Write a coaching explanation for every question

This is the step that actually teaches, not just scores — the same job `/flashcards`' Step 5 does per-question during a live quiz, applied here to the exam's full question set instead. A pattern-level "you keep reaching for hardcoded rules" paragraph in Step 3 is not a substitute for this — it's the cross-question synthesis, this is the per-question teaching. Both are required; neither replaces the other.

For every question, right or wrong:

1. **Wrong answers** lead with the correct option itself, quoted verbatim (not "option B") — never leave the right answer implicit. Then 2-4 sentences: what the correct concept actually is and why it's correct, *and* specifically why the picked option's reasoning fails (not just "that's the wrong pattern"). If the transcript shows real-time reasoning for this question (talked out of a correct instinct, a snap pick, a specific misreading), fold that in — it sharpens the *why*, same as it does for Step 3's patterns. If the miss is an instance of a pattern named in Step 3 or an existing survival-guide heuristic, name that pattern in addition to the concept explanation, not instead of it.
2. **Correct answers** get a lighter touch — one sentence on why the chosen option is the right call. Doesn't need transcript evidence or pattern-naming; the point is reinforcement, not analysis.

This produces one coaching note per question across the full exam (e.g. 60 for a 60-question exam) — expect this to be the longest section of the output. Draft these while Step 2's raw extraction is fresh (same pass or immediately after) rather than reconstructing them later from the Step 3 summary, since the summary necessarily compresses detail this step needs to keep.

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

## Step 3b: Check for cross-cutting reasoning heuristics

Some failure patterns from Step 3 are single-topic gaps (a specific mechanic wasn't known) — those just need a chapter/gap note refresh, handled by `/flashcards import`. Others are general disambiguation rules that cut across domains and topics: a recurring choice between two plausible-sounding fixes where the "why one beats the other" reasoning would apply to a completely different scenario too. The latter belong in `2ndbrain/exam-prep/study-guide-weak-spots/notes/exam-prep-survival-guide.md`, not a topic note — that file is the running collection of these.

A pattern qualifies as cross-cutting if you could state the disambiguation rule *without naming the scenario* (e.g. "does this system already have a piece whose job this is — fix that piece; or is the capability genuinely missing — then add the standard mechanism") and it would still make sense applied to an unrelated scenario. A pattern that's really just "didn't know X mechanic exists" doesn't qualify — that's a knowledge gap, not a disambiguation rule.

If Step 3 surfaced a pattern like this:

1. Read `2ndbrain/exam-prep/study-guide-weak-spots/notes/exam-prep-survival-guide.md` if it exists (it may not, on a first run).
2. If an existing heuristic section already covers this disambiguation (check by substance, not exact wording — e.g. try 4 and try 5 both hit "reaching for more machinery instead of the intended fix"), **add this attempt's misses as new worked examples** to that section rather than creating a duplicate. Recurrence across attempts is itself useful evidence — note it.
3. If it's a genuinely new disambiguation not yet in the file, draft a new section following the existing structure (trap description, the underlying distinction, a one-line test, worked examples with links to the source questions/attempts) and append it. Create the file (with its top-level `# CCAF exam survival guide` header and intro) if it doesn't exist yet.
4. Treat what you write as a **draft**, not a final answer — say so in the Step 4 report. The heuristics that have held up best so far came from Bruno pushing back on a first-pass explanation and the two of you converging on a sharper distinction (see the file's existing sections for the pattern), not from a one-shot write. Flag the drafted/updated heuristic in chat so he can push back the same way, rather than treating it as settled.

Don't force this step — most attempts may surface zero qualifying patterns, and that's fine. Don't lower the bar just to produce a heuristic.

## Step 4: Report in chat

Give Bruno, in plain conversational text (no giant tables — he already saw the domain breakdown in the PDF itself): the overall score/pass-fail in one line (this is the number closest to real CCAF readiness, since the exam covers this material broadly, not just the 4 official courses); 2-4 named failure patterns, each with what it is, why it happens (per the explanations, sharpened by the transcript's real-time reasoning when available), and which questions demonstrate it; 1-2 named strength patterns; and any questions that didn't fit a pattern (the "other misses" bucket), briefly. If Step 2b found any gap-topics matches, mention them briefly too (a note may be worth a quick read/update if this attempt revealed something new about it). If Step 3b drafted a new heuristic or updated an existing one in the survival guide, say so explicitly and flag it as a draft worth pushing back on, not a settled conclusion. If the transcript had any `[Claude]`-tagged Next-steps items (see Step 1), report what each one found — don't let these silently become a footnote or get dropped. Don't paste Step 2a's full per-question coaching into the chat reply — that's what the saved note is for; point to it instead.

## Step 5: Save the analysis note

Write the same report to `2ndbrain/exam-prep/attempts/<source-slug>-<date>-analysis.md` (same slug/date as the source file, `-analysis` suffix) so patterns are comparable across multiple exam attempts over time. Use the source file's own name to derive `<source-slug>-<date>` if it already follows that convention; otherwise ask Bruno for a slug.

Include a `## Answer-by-answer coaching` section: every question in exam order, each as its own entry — question text (trimmed if very long), the option picked, the correct option, and the Step 2a coaching explanation. This is the section Bruno actually studies from question-by-question, same role as `/flashcards`' per-session results file — don't compress or summarize it away in favor of just the pattern-level sections above.

Include a `## Course-topic performance` section as a markdown table, grouped by chapter note (not one row per question — same style as `## Domain breakdown`): columns `Chapter note` (linked), `Score` (`X/Y (Z%)`, with `— weakest`/`— strongest` on the extremes same as the domain table), `Wrong` (the missed question numbers, or `—` if none).

Include a `## Gap-topic performance` section in the same grouped style (only if Step 2b found any matches): `Gap-topic note` (linked), `Score`, `Wrong`. Note in a line above the table that this is informational routing (which existing gap-topics note this maps to), not a separate readiness metric — these questions count toward the overall score like any other, since gap topics are real CCAF exam scope.

If Step 3b touched the survival guide, add a one-line pointer to it (e.g. under the relevant failure pattern or in its own short note) linking to `../study-guide-weak-spots/notes/exam-prep-survival-guide.md`, so the cross-reference is discoverable from this dated attempt file later.

If Step 1 found `[Claude]`-tagged Next-steps items, add a `## Follow-ups from the meeting's Next steps` section documenting what was investigated/computed and the result — same bar as everything else in this file: a finding worth having on record for later, not a scratch note.

## Step 6: Recommend the next step

Point Bruno to `/flashcards import <path-to-original-exam-file>` as the next action — don't run it automatically, since this skill's job ends at analysis.

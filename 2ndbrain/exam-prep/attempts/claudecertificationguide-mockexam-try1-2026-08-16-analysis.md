# Exam analysis — claudecertificationguide mock exam, try 1 (2026-08-16)

Source: `claudecertificationguide-mockexam-try1-2026-08-16.pdf`
Transcript: `claudecertificationguide-mockexam-try1-2026-08-16-transcript.md` (Gemini transcript of Bruno taking the exam out loud)

## Score

653/1000 — **Not passed** (pass mark 720). 39/60 correct (65%).

**Course-mapped accuracy: 22/31 (71%)** — correct rate restricted to the 31 questions that map to an actual chapter note in the 4 completed courses; closest predictor of real CCA-F readiness, since the actual exam is scoped to the official learning path only. Slightly better than the raw 65%, meaning genuine course-material performance is a bit stronger than the overall score suggests (the other 29 questions test material outside the official learning path, which the real exam won't cover).

## Domain breakdown

| Domain | Score |
|---|---|
| D2 Tool Design & MCP Integration | 4/11 (36%) — weakest |
| D4 Prompt Engineering & Structured Output | 6/12 (50%) |
| D5 Context Management & Reliability | 8/11 (73%) |
| D1 Agentic Architecture & Orchestration | 11/14 (79%) |
| D3 Claude Code Configuration & Workflows | 10/12 (83%) — strongest |

## Failure patterns

1. **A small set of "default fixes" gets reached for reflexively, overriding correct in-the-moment reasoning.** The transcript shows this is not a knowledge gap — in several cases Bruno said the correct answer out loud, then talked himself out of it. Defaults to "add a second model/verification pass" for any inconsistency problem (Q14, Q17) and "add few-shot examples" for any tool-confusion problem (Q50, Q52), regardless of whether the actual root cause is a vague spec, a keyword collision, or wrong tool granularity. Q17 is the clearest case: said *"I would replace the pro[se] severity description with concrete code examples"* (the correct answer), then second-guessed it over a polyglot-codebase tangent and picked "second model pass" anyway. Q33: read the correct answer (*"rewrite them to use non-overlapping terms"*) and explicitly rejected it for *"add more detailed descriptions"* instead.

2. **Attraction to solutions that add new machinery over reusing an existing mechanism.** Q16: said *"maintain stratified random set"* (correct) mid-sentence, then got excited about a document-format classifier instead — *"I love this, I love this"* — and switched to the wrong answer. Q1: same instinct (routing classifier over role-scoped subagent split). Q53 (hooks-vs-prompts): reasoned toward "not really about consistency" correctly, then picked "convert everything to hooks for maximum reliability" over risk-matched enforcement anyway.

3. **Reaching for a shell workaround instead of the specific native tool feature.** Q44 and Q59: picked Bash+`sed` over Edit's `replace_all` and Edit's documented context-expansion recovery path — a straightforward knowledge gap (no hesitation in the transcript), not a talked-out-of-it case.

4. **`tool_choice` forcing is known but inconsistently applied.** Nailed the force-then-switch-to-auto pattern on Q45 with full confidence, but missed the identical mechanism on Q43 just beforehand, and on Q20 explicitly said *"I'm not sure though"* before guessing wrong. Inconsistent recall under time pressure, not a concept gap.

5. **Misattributing where context breaks in multi-agent systems.** Q48 and Q55 both blame the downstream/receiving agent when the root cause was the coordinator failing to pass structured context forward — quick, confident, wrong both times, no hesitation in the transcript (a real gap, not a second-guess).

## Strengths

- **Claude Code Configuration & Workflows (83%)** — CLAUDE.md hierarchy, hook ordering (Pre vs Post), `-p`/`--output-format`, session isolation in CI pipelines all handled correctly.
- **Agentic Architecture (79%)** — parallel-vs-sequential orchestration and hub-and-spoke isolation principles solid, outside the two context-passing misses in pattern 5.
- **Reasoning from principle over recall.** Q60: trusted the right architectural instinct (path-scoped `.claude/rules/`) even while saying out loud *"I don't remember learning about `.claude/rules/`"* — good exam-day judgment when memory doesn't have the specific fact.

## Other misses (no clear pattern)

- Q25 (Pydantic validators don't catch cross-field business rules — jumped straight to "temperature/hallucination," confident and quick, a real knowledge gap)
- Q15 (retry-boundary, select-2 — partial credit likely)
- Q21 (didn't immediately honor an explicit "connect me to a human" request — flavor of pattern 1, but a single instance, decided quickly with little deliberation)

## Course-topic performance

The 31 questions that map to actual course material (see course-mapped accuracy above), broken down by chapter note.

| Chapter note | Score | Wrong |
|---|---|---|
| [agents-and-workflows.md](../../claude-api/agents-and-workflows/agents-and-workflows.md) | 10/13 (77%) | Q01, Q48, Q55 |
| [prompt-engineering-techniques.md](../../claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md) | 3/6 (50%) | Q14, Q17, Q33 |
| [automating-and-verifying-work.md](../../claude-code-in-action/automating-and-verifying-work.md) | 4/5 (80%) | Q53 |
| [sharing-and-scaling-claude-code.md](../../claude-code-in-action/sharing-and-scaling-claude-code.md) | 3/3 (100%) | — |
| [introducing-tool-use.md](../../claude-api/tool-use-with-claude/introducing-tool-use.md) | 0/2 (0%) — weakest | Q50, Q52 |
| [long-sessions-and-steering.md](../../claude-code-in-action/long-sessions-and-steering.md) | 1/1 (100%) | — |
| [claude-md-load-order.md](../../claude-code-agent-skills/claude-md-load-order.md) | 1/1 (100%) | — |

Q47's match to `automating-and-verifying-work.md` is conceptual (independent-verification principle) rather than exact — the question's own explanation cites model-tier capability, not session isolation specifically.

## Gap-topic performance

Informational only — separate from the scored domain breakdown above, since gap topics are outside the actual certification exam's scope (see `exam-prep/CLAUDE.md`). This tracks how this attempt did on questions that map to an existing `gap-topics/` note.

| Gap-topic note | Score | Wrong |
|---|---|---|
| [aggregate-metrics-trap.md](../gap-topics/structured-data-extraction/aggregate-metrics-trap.md) | 2/2 (100%) | — |
| [confidence-calibration.md](../gap-topics/structured-data-extraction/confidence-calibration.md) | 1/1 (100%) | — |
| [retry-with-error-feedback.md](../gap-topics/structured-data-extraction/retry-with-error-feedback.md) | 2/3 (67%) | Q15 |
| [risk-based-human-review.md](../gap-topics/structured-data-extraction/risk-based-human-review.md) | 1/1 (100%) | — |
| [semantic-vs-schema-validation.md](../gap-topics/structured-data-extraction/semantic-vs-schema-validation.md) | 0/1 (0%) — weakest | Q25 |
| [stratified-sampling.md](../gap-topics/structured-data-extraction/stratified-sampling.md) | 1/2 (50%) | Q16 |
| [structured-claim-source-mapping.md](../gap-topics/structured-data-extraction/structured-claim-source-mapping.md) | 2/2 (100%) | — |
| [tool-choice-forcing.md](../gap-topics/tool-use-with-claude/tool-choice-forcing.md) | 1/3 (33%) — weakest | Q20, Q43 |

## Practical implication

Patterns 1 and 2 are not recall problems — flashcard drilling on facts won't fix them, since the correct fact was already surfacing in the internal monologue both times. What would help is a beat of "does this option's mechanism actually match the stated root cause?" before committing, especially when the first instinct already named the right one.

## Next step

Run `/flashcards import 2ndbrain/exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16.pdf` to seed weak-spots and generate a study guide.

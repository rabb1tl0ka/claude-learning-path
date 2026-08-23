## Overall result

**895/1000 — PASSED** (pass mark 720). 54 of 60 correct (90%). This raw score is the number closest to real CCAF readiness (per `exam-prep/CLAUDE.md`) — the course/gap mapping below is routing info, not a competing readiness metric. This is Bruno's best try-N result to date (try 3 was 826/1000, 83%).

## Domain breakdown

| Domain | Score | Note |
|---|---|---|
| D1 Agentic Architecture & Orchestration | 12/14 (86%) | |
| D2 Tool Design & MCP Integration | 9/11 (82%) | — weakest |
| D3 Claude Code Configuration & Workflows | 10/12 (83%) | |
| D4 Prompt Engineering & Structured Output | 12/12 (100%) | — strongest (tied) |
| D5 Context Management & Reliability | 11/11 (100%) | — strongest (tied) |

A Gemini transcript of Bruno narrating his live reasoning through the whole attempt is available and sharpens the "why" behind several of these misses beyond what the exam's own explanations show — see inline notes below. Caveat: this transcript's auto-transcription quality is rougher than usual (garbled words throughout, and Gemini's own spoken question numbers drift by ±1 in a couple of places), so quotes below are matched to questions by content, not by the number Bruno says out loud.

## Failure patterns

**1. Reaching for more machinery instead of the intended structural/judgment fix (Q26, Q30, Q32)** — the biggest pattern, 3 of 6 misses. In each case the scenario called for either restoring an existing correct design, switching from a hardcoded rule to model judgment, or invoking a specific process discipline — and Bruno instead added more mechanism on top of the flawed setup:

- **Q30**: a moderation system's hardcoded decision tree (`classify_content` → fixed action) was missing satire and catching sophisticated spam. Bruno's answer added *more granular category labels* to make the tree finer-grained; the correct fix was to replace the hardcoded tree with model-driven judgment entirely — "a hardcoded decision tree cannot adapt to nuance... no matter how fine the labels are." This is the textbook "treating a hardcoded rule as adequate when the question calls for model-driven judgment" trap.

  **Transcript shows this is a genuine "talked himself out of the correct answer" case, not a blind miss.** Live reasoning: "I would go [with] model driven decisions but still use [the] tool... I'm afraid that option B might imply that the tool is not being used... kind of leaning out of B because I feel like the answer implies that the tool won't be used. I think option A is close." He had the correct answer (B) and rejected it based on a misreading — thinking "model-driven decisions" meant abandoning `classify_content` entirely, rather than using the tool's output as input to model judgment instead of a fixed action mapping. He defaulted to A (more granular labels) instead. At the end of the exam he flagged Q30 for revisit, said "I think it's good," and kept the wrong answer — re-confirming the error without re-examining the reasoning that produced it.

- **Q26**: a text classifier had been overloaded with image-analysis responsibilities (2,500-token prompt, 12 tools), and accuracy dropped. Bruno's answer split it into *three* narrower specialists; the correct fix was simpler — restore the original two-agent isolation (text classifier back to text-only, image analysis kept separate) rather than over-splitting further. Transcript: he correctly diagnosed the symptom ("too many tools, man") but jumped straight to "the command agent specialist" (further splitting) without considering that the fix was reverting to the prior, already-correct two-agent design.

- **Q32**: a team updating 25 microservices hit terminology inconsistency after ad-hoc direct execution. Bruno's answer added a discovery step (Explore subagent) before batching changes; the correct fix was Plan Mode — setting an explicit update strategy and terminology *before* any execution, which discovery alone doesn't solve. (Not narrated clearly in the transcript — this question passed without much audible deliberation.)

The common thread: when the real defect is "this needs a different kind of decision-making" (model judgment, or a planning discipline) or "this needs restoring to a simpler prior-correct state," the instinct was to add more structure (finer splits, finer labels, an extra tooling step) instead.

**2. Built-in tool/config mechanics knowledge gaps (Q35, Q50)** — two misses that weren't reasoning failures but missed a specific mechanic:

- **Q35**: renaming a variable across 12 occurrences in a file — Bruno picked "Grep to find them, then Edit 12 times individually" instead of the Edit tool's `replace_all` parameter, which does exactly this in one call. Transcript shows him explicitly ruling out the Bash/`sed` option ("not bash... for sure") but the deliberation doesn't surface `replace_all` as a considered option at all — consistent with a recall gap (the mechanic wasn't on his radar to weigh), not a reasoning error.

- **Q50**: a polyglot codebase needing per-file-type conventions — Bruno picked a single `.claude/rules/infrastructure.md` file with glob `["**/*"]`, not registering that `**/*` matches *every* file regardless of type, which defeats the whole point of conditional loading. The correct answer was three separate path-scoped rule files, one per file type. Transcript: "the one that threw me off was having a CLAUDE.md in the project root for docker conventions... actually could be [dockerfiles] scattered throughout various service directories... it's [.claude/]rules." He correctly reasoned his way to needing a path-scoped rules mechanism instead of a root CLAUDE.md, but didn't catch that his own answer's glob pattern silently made it behave exactly like the root-CLAUDE.md option he'd just ruled out.

**3. Misreading a question's actual defect and solving a different problem (Q54)** — the transcript catches something the exam's own explanation can't show. For the payment-refund tool question, Bruno's real-time reasoning was: "the big problem here is the agent calling a payment processing tool to check for a refund... you should call the refund tool, not a payment processing tool. But anyway, there's no option for this, I think it's C." He diagnosed a *tool-selection* problem (wrong tool called) that isn't what the scenario describes or what any answer choice addresses, then picked C (blame the tool's response format) as the closest fit to that invented narrative — rather than re-reading the scenario for what it actually says (a valid empty result being misread as a failure by the agent's own recovery logic, the same "access failure vs. empty result" distinction he answered correctly seven other times this exam). This wasn't a concept gap; it was answering a self-generated version of the question instead of the one on the page.

## Strength patterns

**Prompt engineering & structured output (D4) and Context management & reliability (D5) — both 100%.** Multi-source conflict handling (Q4, Q12 — annotate both values with source/date rather than averaging or discarding one), error-propagation structured metadata (Q21, Q33, Q44, Q45, Q48 — distinguishing transient/business/validation error categories and access-failure vs. valid-empty-result), and context degradation mitigations (Q36, Q37, Q38, Q56 — lost-in-the-middle, scratchpad persistence, progressive-summarisation data loss) were all answered correctly, including several repeats of the same underlying concept across different scenarios — a sign this material is genuinely internalized, not just pattern-matched once.

## Other misses

None remaining — all 6 misses are covered by the two failure patterns above (Q54 belongs to pattern 3, not a separate one-off, once the transcript is factored in: the same "access failure vs. valid empty result" distinction shows up correctly answered in Q2, Q21, Q33, Q44, Q45, Q48, and Q59, so the miss traces to a misread of this specific scenario rather than a gap in the concept).

## Coverage-gap note (informational)

18 of 60 questions test material that doesn't map to any of the 4 course notes or an existing gap-topics note — these are genuine CCAF scope the courses don't teach. Bruno went 16/18 (89%) on these, roughly matching his overall score, so this isn't a hidden weak spot — but worth naming as clusters if a research note is ever wanted:
- **MCP/tool structured error responses** (error categories, transient vs. business vs. validation, access-failure vs. empty-result, silent suppression) — Q2, Q21, Q33, Q44, Q45, Q48, Q54, Q59 (7/8 correct, only Q54 missed).
- **Context window degradation & management** (lost-in-the-middle, scratchpad mitigation, progressive summarisation, position effects) — Q9, Q36, Q37, Q38, Q56 (5/5 correct).
- **Message Batches API** (batch failure handling, batch-vs-realtime tradeoffs) — Q10, Q40 (2/2 correct).
- **Claude Code path-specific `.claude/rules/` files** — Q50, Q52 (1/2 correct — see Q50 above).
- **Claude Code Grep-vs-Glob built-in tool choice** — Q53 (1/1 correct).

## Course-topic performance

Grouped by chapter note. Routing info from the exam's tested concepts matched to existing course notes by subject matter, not by the exam's own third-party lesson labels.

| Chapter note | Score | Wrong |
|---|---|---|
| [text-editor-tool.md](../../ccaf-learning/claude-api/tool-use-with-claude/text-editor-tool.md) | 0/1 (0%) — weakest | Q35 |
| [long-sessions-and-steering.md](../../ccaf-learning/claude-code-in-action/long-sessions-and-steering.md) | 2/3 (67%) | Q32 |
| [agents-and-workflows.md](../../ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md) | 8/10 (80%) | Q26, Q30 |
| [automating-and-verifying-work.md](../../ccaf-learning/claude-code-in-action/automating-and-verifying-work.md) | 7/7 (100%) — strongest | — |
| [introducing-tool-use.md](../../ccaf-learning/claude-api/tool-use-with-claude/introducing-tool-use.md) | 1/1 (100%) | — |
| [claude-md-load-order.md](../../ccaf-learning/claude-code-agent-skills/claude-md-load-order.md) | 1/1 (100%) | — |
| [what-are-skills.md](../../ccaf-learning/claude-code-agent-skills/what-are-skills.md) | 1/1 (100%) | — |
| [mcp-overview.md](../../ccaf-learning/claude-api/model-context-protocol/mcp-overview.md) | 1/1 (100%) | — |
| [prompt-engineering-techniques.md](../../ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md) | 3/3 (100%) | — |
| [structured-data.md](../../ccaf-learning/claude-api/accessing-claude-with-the-api/structured-data.md) | 2/2 (100%) | — |
| [sharing-and-scaling-claude-code.md](../../ccaf-learning/claude-code-in-action/sharing-and-scaling-claude-code.md) | 3/3 (100%) | — |
| [prompt-caching.md](../../ccaf-learning/claude-api/features-of-claude/prompt-caching.md) | 1/1 (100%) | — |
| [using-multiple-tools.md](../../ccaf-learning/claude-api/tool-use-with-claude/using-multiple-tools.md) | 1/1 (100%) | — |

## Gap-topic performance

Informational routing only — which existing gap-topics note this maps to, not a separate readiness metric. These questions count toward the overall score like any other, since gap topics are real CCAF exam scope.

| Gap-topic note | Score | Wrong |
|---|---|---|
| [structured-claim-source-mapping.md](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md) | 3/3 (100%) | — |
| [stratified-sampling.md](../../ccaf-learning/structured-data-extraction/stratified-sampling.md) | 1/1 (100%) | — |
| [tool-choice-forcing.md](../../ccaf-learning/tool-choice-forcing/tool-choice-forcing.md) | 1/1 (100%) | — |
| [retry-with-error-feedback.md](../../ccaf-learning/structured-data-extraction/retry-with-error-feedback.md) | 1/1 (100%) | — |
| [semantic-vs-schema-validation.md](../../ccaf-learning/structured-data-extraction/semantic-vs-schema-validation.md) | 1/1 (100%) | — |

## Next step

Run `/flashcards import exam-prep/attempts/claudecertificationguide-mockexam-try4-2026-08-22.pdf` from `2ndbrain/` to seed the missed questions into weak-spot tracking and generate a targeted study guide.

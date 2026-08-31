# Try 6 mock exam — failure-pattern analysis

Source: `claudecertificationguide-mockexam-try6-2026-08-23.pdf`. No transcript provided for this attempt.

**Score: 901/1000 — PASSED (54/60 correct, 90%; pass mark 720).** This is the readiness number that matters — the CCAF exam scope is Claude Code + Agent SDK + Claude API + MCP broadly, not just the 4 Learning Path courses, so course-mapped accuracy below is routing information, not a competing score.

## Domain breakdown

| Domain | Score | Notes |
|---|---|---|
| D1 Agentic Architecture & Orchestration | 14/14 (100%) | strongest |
| D2 Tool Design & MCP Integration | 8/11 (73%) | weakest |
| D3 Claude Code Configuration & Workflows | 10/12 (83%) | |
| D4 Prompt Engineering & Structured Output | 11/12 (92%) | |
| D5 Context Management & Reliability | 11/11 (100%) | strongest |

## Failure patterns

### 1. Edit tool non-unique match — reaching for `sed`/Bash instead of the tool's own recovery path (Q03, Q11)

Both questions describe Claude Code's Edit tool failing with a "non-unique match" error. Both times the answer picked was "switch to Bash with `sed`" — bypassing the Edit tool's safety guarantees and risking line-number/special-character fragility. The correct move both times stays inside the Edit tool: expand the search string with more surrounding context (Q03), or use Grep first to identify the correct occurrence, then expand context (Q11).

This is not a one-off — it's the third attempt to hit this exact reflex (try 1 Q44/Q59, try 4 Q35, now try 6 Q03/Q11), always the same shape: a built-in tool has a documented recovery step, and the pick is a generic shell workaround instead. Added as new worked examples to the "Reach for the native tool feature before a generic workaround" heuristic in the survival guide — flagged as a draft update, not settled, since it was written from the explanation text alone (no transcript to confirm real-time reasoning this time).

### 2. Tool granularity — patching a tool's description instead of splitting it (Q59)

A single tool (`analyze_content`) was doing three unrelated jobs (web scraping, document parsing, code analysis) with an under-specified description. The pick was "improve the description to list supported content types" — the correct fix is to split the tool into purpose-specific tools (`extract_web_results`, `parse_document`, `analyze_code`). This is a near-exact repeat of try 2 Q11, and matches the existing "Tool/agent-granularity direction" heuristic's split rule (3+ genuinely unrelated purposes on one tool → split, not describe harder). Added as a new worked example there — also a draft, worth a second look.

### Other misses (no sibling this attempt)

- **Q02** (tool_choice forcing) — picked `tool_choice: 'any'` (guarantees *a* tool call, but not the *right* one when multiple tools exist) instead of forcing the specific tool by name. Single-topic mechanic, not a reasoning pattern.
- **Q12** (`.claude/rules/` path-specific glob activation, select 2) — one of the two correct options was missed in favor of an "always-on loading" framing that's actually the anti-pattern the question is testing against.
- **Q48** (sequencing dependent vs. independent fixes) — three issues where issue 3 (schema field name) changes the output shape issues 1 and 2 must conform to; picked the interview pattern (for unfamiliar domains) over the correct move (fix the dependency first, then batch the independent ones). This didn't recur elsewhere this attempt, so it's logged here rather than drafted as a new cross-cutting heuristic — worth watching for a second occurrence before promoting it.

## Strengths

D1 (Agentic Architecture & Orchestration) and D5 (Context Management & Reliability) both went 100%. Coordinator/subagent responsibilities, hub-and-spoke routing, task decomposition (fixed pipeline vs. dynamic adaptive vs. multi-pass), error propagation (silent suppression, structured error context), and information provenance/conflict handling were all solid across a wide spread of questions (14 and 11 questions respectively) — this isn't a narrow lucky streak, it's genuine command of the multi-agent design space.

## Course-topic performance

Judged by actual tested concept against existing chapter notes, not the exam's own lesson citations.

| Chapter note | Score | Wrong |
|---|---|---|
| [agents-and-workflows.md](../../ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md) | 15/15 (100%) — strongest | — |
| [long-sessions-and-steering.md](../../ccaf-learning/claude-code-in-action/long-sessions-and-steering.md) | 6/6 (100%) | — |
| [automating-and-verifying-work.md](../../ccaf-learning/claude-code-in-action/automating-and-verifying-work.md) | 3/3 (100%) | — |
| [prompt-engineering-techniques.md](../../ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md) | 2/2 (100%) | — |
| [what-are-skills.md](../../ccaf-learning/claude-code-agent-skills/what-are-skills.md) | 2/2 (100%) | — |
| [mcp-overview.md](../../ccaf-learning/claude-api/model-context-protocol/mcp-overview.md) | 3/3 (100%) | — |
| [introducing-tool-use.md](../../ccaf-learning/claude-api/tool-use-with-claude/introducing-tool-use.md) family (stop_reason, tool distribution) | 3/3 (100%) | — |

Questions counted: Q06, Q13, Q16, Q19, Q23, Q24, Q33, Q36, Q39, Q40, Q41, Q46, Q47, Q50, Q53, Q54, Q55, Q56, Q57, Q07, Q08, Q10, Q25, Q28, Q31, Q32. Some overlap between agents-and-workflows.md and D1 orchestration questions is intentional — that chapter covers the section broadly.

## Gap-topic performance

Informational routing only — where an existing gap-topics note (self-directed research the 4 courses don't teach) maps to a missed or hit concept. These questions count toward the overall score like any other; gap topics are real CCAF scope.

| Gap-topic note | Score | Wrong |
|---|---|---|
| [tool-choice-forcing.md](../../ccaf-learning/tool-choice-forcing/tool-choice-forcing.md) | 1/2 (50%) | Q02 |
| [aggregate-metrics-trap.md](../../ccaf-learning/structured-data-extraction/aggregate-metrics-trap.md) | 1/1 (100%) | — |
| [retry-with-error-feedback.md](../../ccaf-learning/structured-data-extraction/retry-with-error-feedback.md) | 3/3 (100%) | — |
| [stratified-sampling.md](../../ccaf-learning/structured-data-extraction/stratified-sampling.md) / [risk-based-human-review.md](../../ccaf-learning/structured-data-extraction/risk-based-human-review.md) | 1/1 (100%) | — |
| [structured-claim-source-mapping.md](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md) | 2/2 (100%) | — |
| [semantic-vs-schema-validation.md](../../ccaf-learning/structured-data-extraction/semantic-vs-schema-validation.md) | 1/1 (100%) | — |

A sizeable chunk of D1/D2/D3/D4 questions this attempt (multi-agent error propagation, information-provenance conflict handling, path-specific `.claude/rules/` glob activation, Edit-tool non-unique-match recovery, nullable/optional schema fields, attention dilution) don't map to any existing course or gap-topics note — logged as genuine coverage gaps rather than forced into a bad match, per this repo's coverage-gap convention. Worth a research pass if you want dedicated notes, but none were force-fit here.

## Survival guide updates (draft — push back if these don't hold up)

Two existing heuristics in [`exam-prep-survival-guide.md`](../study-guide-weak-spots/notes/exam-prep-survival-guide.md) got new worked examples from this attempt, not new sections:

- **"Reach for the native tool feature before a generic workaround"** — Q03 and Q11 added; this is now a 3-attempt-spanning pattern (try 1 → try 4 → try 6), always the Edit tool vs. `sed`/Bash.
- **"Tool/agent-granularity direction"** — Q59 added under the "split" examples, near-identical to try 2 Q11.

Both were written from the exam's own explanation text (no transcript this run to sharpen the "why") — treat them as first drafts.

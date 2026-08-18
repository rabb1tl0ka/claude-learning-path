# Exam analysis — claudecertificationguide mock exam, try 2 (2026-08-18)

Source: `claudecertificationguide-mockexam-try2-2026-08-18.pdf`
No transcript provided for this attempt.

## Score

744/1000 — **Passed** (pass mark 720). 45/60 correct (75%).

**Course-mapped accuracy: 14/20 (70%)** — correct rate restricted to the 20 questions that map to actual course material across the 4 completed courses; closest predictor of real CCA-F readiness, since the actual exam is scoped to the official learning path only. Slightly below the raw 75%, and notably lower than try 1's 71% course-mapped rate — this exam attempt leaned harder on Claude-Code-specific multi-agent mechanics (subagent `Task` tool, `fork_session`, worktree merge sequencing, MCP config scoping, structured error taxonomies, `tool_choice` forcing, Pydantic validation, stratified sampling) that the 4 official courses simply don't teach in this depth. Only 20 of 60 questions map to material the official learning path actually covers.

## Domain breakdown

| Domain | Score |
|---|---|
| D2 Tool Design & MCP Integration | 7/11 (64%) — weakest |
| D1 Agentic Architecture & Orchestration | 10/14 (71%) |
| D3 Claude Code Configuration & Workflows | 9/12 (75%) |
| D4 Prompt Engineering & Structured Output | 9/12 (75%) |
| D5 Context Management & Reliability | 10/11 (91%) — strongest |

## Failure patterns

1. **Reaches for a workaround or manual process instead of the specific native mechanism already built for the job.** Q2: picked `fork_session` to spawn two parallel subagent branches, when the correct mechanism is simply emitting multiple `Task` tool calls in a single coordinator response — `fork_session` is for divergent exploration from a shared baseline, not concurrent subagent invocation. Q39: picked Glob-then-Read-then-Edit to find every reference to a renamed API endpoint, when Grep-then-Edit is the direct, minimal path — reading every doc file to search for a string wastes context that Grep's content search avoids. Q49: kept `tool_choice: 'any'` instead of forcing the specific extraction tool by name — `any` still lets the model pick among multiple tools, when the precise fix (force by name) eliminates ambiguity entirely. Q57: reached for a system prompt instruction telling the agent to prefer the MCP tool over Bash, when the sustainable fix is enhancing the tool's own sparse description — a brittle per-agent patch instead of a fix that scales to every agent and context. Q58: wrote a `.claude/rules/` file banning mocks outright, when the actual gap was underspecified CLAUDE.md criteria (real DB connections, contract assertions) — over-broad enforcement instead of a precise standard. Q33: assumed hooking a nonexistent "Compact" tool via PreToolUse, not knowing `PreCompact` is its own dedicated lifecycle hook event.

2. **Gets tool-granularity direction backwards.** Q11: a tool doing 3 distinct jobs (web scraping, document parsing, code analysis) needed splitting into purpose-specific tools, but the pick was "improve the description" — patching the symptom (ambiguous when-to-use) rather than fixing the actual cause (one tool serving too many use cases). Q24: 22 near-duplicate tools needed *consolidating* down to ~4, but the pick was splitting them across two MCP servers — server boundaries are invisible to the agent (all connected tools appear in one flat list), so this doesn't reduce the agent's real choice set at all. Same underlying miss in both directions: not recognizing which way tool count needs to move to fix the actual selection problem.

3. **Chooses more validation/detection machinery over the simpler fix that prevents the problem at the source.** Q3 and Q48 (same root issue, missed twice): fields fabricate data when the source document doesn't contain them; the fix is making those fields nullable so the model can return `null` instead. Both times, the pick added infrastructure instead — separate schemas per document type (Q3), a post-hoc validation step (Q48) — rather than removing the schema-level pressure to fabricate. Q16: picked prose footnotes citing sources, when the team had already experienced footnotes degrading through editing passes — structured provenance metadata (survives editing because it's a data field, not prose) was the direct, already-flagged-as-fragile-alternative fix.

4. **Misattributes the root cause of a multi-agent failure to the wrong agent.** Q14: blamed the synthesis agent's system prompt for missing citations, when the synthesis agent never received the source URLs/document names in the first place — the coordinator stripped structured metadata before passing content along. Q59: blamed the document-analysis subagent's source access for a report that only covered solar/wind out of six renewable categories, when the coordinator's own task decomposition never assigned anyone to research geothermal/tidal/biomass/fusion. Same shape as try 1's context-passing misattribution pattern — confidently blaming the downstream/receiving agent instead of tracing the failure to the coordinator.

## Strengths

- **Context Management & Reliability (91%, strongest domain)** — information provenance and multi-source synthesis genuinely solid: correctly handled conflicting data points by preserving both with full attribution rather than averaging or picking one "more authoritative" source (Q9, Q18, Q56), and correctly identified silent-suppression as the worst error-propagation anti-pattern (Q19).
- **Structured error handling** — beyond the one root-cause misattribution above, the actual error-category taxonomy (transient vs business vs validation, retryable vs not) was applied correctly across several distinct scenarios (Q23, Q28, Q32, Q44, Q45, Q46) — a genuinely internalized mental model, not lucky guessing.
- **CI/CD and worktree fundamentals** — parallel Claude Code instances via `git worktree`, and the general shape of sequencing merges to avoid conflicts, came through correctly in most instances (Q26, Q40), even where the more nuanced merge-sequencing question (Q15) was missed.

## Other misses (no clear pattern)

- Q6 (stale tool results after 40 minutes — picked "re-read the files in the current session" over "fresh session with a summary + re-read"; a session-hygiene miss, not tied to the other four patterns)
- Q15 (two Claude Code instances on separate worktrees needing sequenced merges rather than independent edits + late conflict resolution — related to Q6's state-management theme but not a repeated enough shape to name as its own pattern)

## Course-topic performance

The 20 questions that map to actual course material (see course-mapped accuracy above), broken down by chapter note.

| Chapter note | Score | Wrong |
|---|---|---|
| [agents-and-workflows.md](../../ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md) | 5/5 (100%) — strongest | — |
| [prompt-engineering-techniques.md](../../ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md) | 2/2 (100%) — strongest | — |
| [what-are-skills.md](../../ccaf-learning/claude-code-agent-skills/what-are-skills.md) | 1/1 (100%) — strongest | — |
| [long-sessions-and-steering.md](../../ccaf-learning/claude-code-in-action/long-sessions-and-steering.md) | 5/8 (63%) | Q06, Q15, Q58 |
| [automating-and-verifying-work.md](../../ccaf-learning/claude-code-in-action/automating-and-verifying-work.md) | 1/2 (50%) | Q33 |
| [introducing-tool-use.md](../../ccaf-learning/claude-api/tool-use-with-claude/introducing-tool-use.md) | 0/2 (0%) — weakest | Q11, Q57 |

Several matches here are conceptual rather than exact — e.g. Q22/Q29/Q30/Q31/Q42 map to `agents-and-workflows.md`'s workflow-pattern content (chaining, parallelization, workflow-vs-agent selection) even though the questions frame it in multi-agent/subagent terms the course itself doesn't use; Q6/Q15/Q26/Q40/Q51 map to `long-sessions-and-steering.md`'s worktree/plan-mode/session-state content the same way. `introducing-tool-use.md` being the weakest chapter (0/2) is notable: both misses (Q11, Q57) are exactly the tool-description-quality failure mode that chapter's own notes flagged as a parallel to Skill/Agent descriptions — the concept was already flagged as important and still missed twice.

## Gap-topic performance

Informational only — separate from the scored domain breakdown above and from course-mapped accuracy, since gap topics are outside the actual certification exam's scope (see `exam-prep/CLAUDE.md`).

| Gap-topic note | Score | Wrong |
|---|---|---|
| [semantic-vs-schema-validation.md](../../ccaf-learning/structured-data-extraction/semantic-vs-schema-validation.md) | 2/2 (100%) | — |
| [retry-with-error-feedback.md](../../ccaf-learning/structured-data-extraction/retry-with-error-feedback.md) | 3/3 (100%) | — |
| [aggregate-metrics-trap.md](../../ccaf-learning/structured-data-extraction/aggregate-metrics-trap.md) | 1/1 (100%) | — |
| [stratified-sampling.md](../../ccaf-learning/structured-data-extraction/stratified-sampling.md) | 2/2 (100%) | — |
| [confidence-calibration.md](../../ccaf-learning/structured-data-extraction/confidence-calibration.md) | 1/1 (100%) | — |
| [structured-claim-source-mapping.md](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md) | 1/2 (50%) — weakest | Q16 |
| [tool-choice-forcing.md](../../ccaf-learning/tool-choice-forcing/tool-choice-forcing.md) | 1/2 (50%) — weakest | Q49 |

Compared to try 1 (where `tool-choice-forcing.md` was the weakest gap topic at 1/3), the same note is still a soft spot this attempt — worth a quick re-read given it's shown up as a miss in both attempts.

## Practical implication

Patterns 1 and 2 (reaching for a workaround/extra machinery instead of the precise native fix, and getting tool-granularity direction backwards) account for 8 of the 15 misses and share a root shape: when a scenario names a specific mechanism (a hook event, a tool_choice mode, a splitting-vs-consolidating tradeoff), the fix is almost always "use that exact mechanism precisely," not "add a layer around it." Worth drilling `tool_choice` modes and Claude Code hook event names as flashcards specifically, since both showed up as repeat misses.

## Next step

Run `/flashcards import 2ndbrain/exam-prep/attempts/claudecertificationguide-mockexam-try2-2026-08-18.pdf` to seed weak-spots and generate a study guide.

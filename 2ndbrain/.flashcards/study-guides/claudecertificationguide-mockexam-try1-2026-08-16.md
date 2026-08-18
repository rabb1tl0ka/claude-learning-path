# Study guide — claudecertificationguide mock exam, try 1 (2026-08-16)

Grounded in your own chapter notes. Highest-impact (most-missed) topics first. Each topic marked as grounded in an actual missed question from this import, or in general weak-spot history.

## 1. Coordinator context-passing in multi-agent systems (2 misses — grounded in this import)

**Core theory** (from `ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md`): in a hub-and-spoke architecture, subagents don't share memory or communicate directly — the coordinator is the only thing that sees every subagent's output. If a downstream agent is missing information a sibling already produced, the fault is almost always the **coordinator failing to forward it as context**, not the downstream agent's own prompt or the upstream agent's output quality.

**X vs Y**: "the receiving agent's system prompt needs an instruction" vs. "the coordinator isn't passing the right data forward" — you defaulted to blaming the receiving agent's prompt twice (synthesis-agent citations, test-update agent's stale schema references) when the real fix was upstream, at the coordinator's context-passing step.

**Self-check**: If a synthesis agent drops source attribution even though its inputs were well-sourced, where do you look first — the synthesis agent's prompt, or what the coordinator handed it?

## 2. Prompt clarity vs. adding a verification pass (2 misses — grounded in this import)

**Core theory** (from `prompt-engineering-techniques.md`'s "being specific" and "providing examples"): when output is inconsistent because criteria are vague or written as prose, the fix is making the *spec* concrete — quality/attribute lists, step-by-step instructions, or concrete examples — not adding another pass that re-checks the same vague criteria. A second model pass re-applies the same ambiguous judgment; it doesn't remove the ambiguity.

**X vs Y**: "add a verification/second-pass step" vs. "rewrite the spec to be concrete and checkable." You reached for the verification-pass fix twice (a 40% false-positive review category, inconsistent severity ratings) when the actual root cause was vague prose criteria that needed rewriting into concrete criteria/examples.

**Self-check**: A pipeline is inconsistent because its instructions are vague. Do you add a second pass to catch the inconsistency, or fix the instructions so there's nothing inconsistent to catch?

## 3. Keyword overlap between prompt phrasing and tool names (1 miss — grounded in this import)

**Core theory** (from `prompt-engineering-techniques.md`'s "being clear and direct"): if an instruction's phrasing happens to share words with a tool's name, the model can misroute based on that surface-level keyword match rather than the actual task. The fix is removing the overlap (distinct, non-overlapping terms), not adding more detail to the existing (overlapping) wording.

**Self-check**: your system prompt says "check for security issues in each function" and you also have a tool named `security_check`. If the model starts misapplying that tool to unrelated instructions mentioning "security," is the fix a longer tool description, or renaming/rewording to remove the shared keyword?

## 4. Few-shot examples vs. tool granularity (1 miss — grounded in this import)

**Core theory** (from `agents-and-workflows.md`'s "tools should be abstract, not hyper-specialized" combined with `prompt-engineering-techniques.md`'s few-shot guidance): few-shot examples fix *behavioral* inconsistency (a tool used correctly but inconsistently styled). They don't fix *scope* problems — a tool doing three unrelated jobs badly needs to be **split into purpose-specific tools**, not given more examples about which job to do when.

**Self-check**: a single tool handles web scraping, document parsing, and code analysis, and results are poor across all three. Is the fix better examples, or fewer responsibilities per tool?

## 5. Matching enforcement strictness to actual risk (1 miss — grounded in this import)

**Core theory** (from `ccaf-learning/claude-code-in-action/automating-and-verifying-work.md`'s hooks section): hooks exist for rules Claude *can't be allowed to skip* — safety-critical, hard-to-reverse actions. A recoverable failure (a missed backup on version-controlled files) doesn't need the same deterministic enforcement as an irreversible one (writing outside the project directory). Converting *everything* to a hook "for consistency" adds implementation complexity without matching benefit.

**Self-check**: which guardrail deserves a hook — one where failure is expensive/irreversible, or one where failure is merely inconvenient and recoverable?

---

## Coverage gaps — not in any existing chapter note

These questions don't map to material in this repo's notes. They're real gaps, not weak-spot misses — studying existing notes won't help here; this content needs new research or a new source.

- **Structured Data Extraction / Human-Review-Calibration domain** (largest gap — 11 exam questions across your correct and incorrect answers): aggregate accuracy metrics masking per-document-type performance, confidence-threshold calibration against labelled validation sets, stratified sampling for ongoing monitoring, retry-with-error-feedback vs. naive retry, Pydantic schema validation vs. cross-field business-rule validation, structured claim-source mapping for attribution. None of the 4 completed courses cover this domain at all.
- **Claude Code built-in tools (Grep/Glob/Edit mechanics)**: when to use Grep vs. Glob for a two-step search task, Edit's `replace_all` parameter, and Edit's documented recovery path for a "non-unique match" error (expand the search string's context, or use `replace_all`) are not documented in any chapter note.
- **`.claude/rules/` (path-scoped rule files)**: a mechanism for loading conventions only when matching files are touched, distinct from CLAUDE.md (always loaded) and Skills (invoked by description match). Not covered by any existing chapter note — you correctly reasoned toward it from architectural principle on one question despite not recalling the name, which is a good sign, but it's worth an actual note now that you know it exists (see this session's `/exam-analysis` conversation for a working definition).
- **`tool_choice` forcing mechanics** (`auto` / `any` / forcing a specific tool by name): the Claude API supports forcing a specific tool call by name for one turn, then switching back to `auto`. This isn't documented in any tool-use chapter note, and it's exactly the kind of fact that would resolve the pattern noted in `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md` where this mechanism was known but inconsistently applied.
- **Anthropic Message Batches API `custom_id` field**: matching batch results back to their original request via `custom_id` rather than relying on result ordering. Not covered anywhere.
- **Customer support escalation heuristics**: explicit "connect me to a human" requests should escalate immediately regardless of how simple the underlying issue seems. Not covered by any chapter note (no course section on customer-support-agent design specifically).
- **Iterative refinement technique selection** (batch vs. sequential feedback, when to use the "interview pattern" for domain-unfamiliar work vs. when the developer already knows the requirements): partially adjacent to `long-sessions-and-steering.md`'s plan-mode material, but the specific decision framework isn't written down.
- **Context degradation mitigation for long single-agent sessions** (scratchpad files to persist findings vs. just increasing context window size): not covered — `long-sessions-and-steering.md` covers `/compact` and rewind but not this specific technique.

## Suggested study order

1. Coordinator context-passing (topic 1) — highest-value, appeared twice, and the underlying principle (hub-and-spoke isolation) is otherwise a strength area, so this should be a quick fix.
2. Prompt clarity vs. verification pass (topic 2) — same underlying instinct caused two misses; internalizing "fix the spec, not add a check" should resolve both.
3. Keyword overlap and few-shot-vs-granularity (topics 3-4) — smaller, both live in the same chapter note.
4. Hooks-vs-risk (topic 5).
5. Then tackle the coverage gaps, roughly in order of exam weight: Structured Data Extraction domain first (11 questions is the single biggest gap by far), then `.claude/rules/` and `tool_choice` forcing (both are fast to close — you're one fact away in each case), then the smaller ones.

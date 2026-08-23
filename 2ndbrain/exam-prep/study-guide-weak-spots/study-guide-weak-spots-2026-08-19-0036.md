# Study guide — weak spots (for NotebookLM podcast)

Generated 2026-08-19 from `.flashcards/weak-spots.md` (topics with wrong-rate ≥ 50%, excluding "Coverage gap" topics already tracked separately in `ccaf-learning/`). Curriculum reference material crawled from https://claudecertificationguide.com into `2ndbrain/ccaf-guide/` — see that directory for the full 5-domain crawl.

For each topic: the file set to feed into NotebookLM, plus the specific "why you keep missing this" so the podcast generation actually targets the failure mode, not just the topic label.

## coordinator-context-passing (100% wrong, 4/4)

**Why you keep missing it:** every miss is the same shape — blaming the *receiving* subagent (synthesis agent has no citations, test-update agent references renamed schemas) when the actual root cause is the *coordinator* stripping structured metadata or under-decomposing the task before delegating. Subagents get zero automatic context; they only know what the coordinator explicitly writes into their prompt.

Files:
- `ccaf-guide/1-agentic-architecture/1-2-orchestration-patterns.md` — hub-and-spoke rule, narrow-decomposition failure pattern
- `ccaf-guide/1-agentic-architecture/1-3-subagent-invocation-context.md` — the 3 context-passing rules, structured metadata format, exam traps
- `ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md` — course baseline (workflow patterns, no coordinator-specific mechanics)
- `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md` (pattern 5)
- `exam-prep/attempts/claudecertificationguide-mockexam-try2-2026-08-18-analysis.md` (pattern 4)

## few-shot-vs-tool-granularity (100% wrong, 3/3)

**Why you keep missing it:** the topic label is misleading — every actual missed question is about **tool granularity/scoping** (one `analyze_content` tool doing 3 unrelated jobs with a vague description), not about few-shot examples. The reflexive wrong answer is "improve the description" or "add few-shot examples" when the fix is **splitting into separate, narrowly-scoped tools**. Keep the few-shot and tool-granularity concepts mentally separate — they solve different problems.

Files:
- `ccaf-guide/2-tool-design-mcp/2-1-tool-schema-design.md` — misrouting anti-pattern, why few-shot/consolidation are wrong first fixes, tool-splitting strategy
- `ccaf-guide/2-tool-design-mcp/2-3-tool-distribution-choice.md` — 4-5 tools-per-agent rule, decision matrix for choosing the remedy
- `ccaf-guide/4-prompt-engineering/4-2-few-shot-prompting.md` — has a dedicated section explicitly distinguishing few-shot (in-prompt examples) from tool granularity (tool-scope design)
- `ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md` — course baseline on few-shot only
- `.flashcards/history.jsonl` (grep `few-shot-vs-tool-granularity`) — exact missed question text

## tool-choice/tool-granularity confusion (100% wrong, 2/2)

Files:
- `ccaf-guide/2-tool-design-mcp/2-3-tool-distribution-choice.md` — all 3 `tool_choice` modes (auto/any/forced) with JSON syntax
- `ccaf-learning/claude-api/tool-use-with-claude/introducing-tool-use.md` — course baseline (weakest-scoring chapter, 0/2 on try2)
- `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md` (pattern 4 — force-then-auto pattern known but inconsistently applied)

## worktree-coordination (100% wrong, 2/2)

**Gap found:** confirmed by direct crawl (all 5 domains) and a site search that **claudecertificationguide.com has no page covering worktree merge-coordination** — only a passing mention in the `long-running-sessions` note. Your actual miss (two worktrees both needing to edit the same shared file) needs *sequenced merges*, not "isolate and merge independently" — worktrees isolate the working tree, not the shared file each branch merges back into.

Files:
- `ccaf-learning/worktree-coordination/merge-sequencing.md` — new gap-topic research note (2026-08-19): what worktrees actually isolate vs. share, why isolation ≠ conflict-free, the sequenced-merge fix, sourced from the official Claude Code docs + the missed mock-exam scenario (flagged there as a synthesis, not a direct citation — validate against future sources)
- `ccaf-learning/claude-code-in-action/long-sessions-and-steering.md` — only existing course-backed source, surface-level definition only
- `exam-prep/attempts/claudecertificationguide-mockexam-try2-2026-08-18-analysis.md` (Q15, "Other misses" section) — the actual missed scenario and correct reasoning
- `.flashcards/history.jsonl` (grep `worktree-coordination`) — exact missed question text (two Claude Code instances both modifying `OrderService.java`)

## tool description quality (67% wrong, 2/3)

Files:
- `ccaf-guide/2-tool-design-mcp/2-1-tool-schema-design.md` — the 5 required elements of a production-grade tool description
- `ccaf-learning/claude-api/tool-use-with-claude/introducing-tool-use.md` — course baseline, already flags this as important (and still missed twice per try2 analysis)

## role-scoped-subagent-splitting (50%, 1/2)

Files:
- `ccaf-guide/1-agentic-architecture/1-6-task-decomposition.md` — fixed pipelines vs dynamic decomposition, attention-dilution failure mode
- `ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md`
- `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md` (pattern 2 — attraction to new machinery over role-scoped split)

## prompt-clarity-vs-verification-patch (50%, 1/2)

Files:
- `ccaf-guide/4-prompt-engineering/4-1-system-prompts.md` — explicit criteria vs vague instructions, false-positive trust problem
- `ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md`

## concrete-examples-vs-vague-prose (50%, 1/2)

Files:
- `ccaf-guide/4-prompt-engineering/4-2-few-shot-prompting.md`
- `ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md`

## prompt-keyword-overlap (50%, 1/2)

Files:
- `ccaf-guide/2-tool-design-mcp/2-1-tool-schema-design.md` — system-prompt keyword conflicts overriding tool descriptions
- `ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md`

## hooks-vs-claude-md-enforcement (50%, 1/2)

Files:
- `ccaf-guide/1-agentic-architecture/1-4-workflow-enforcement-handoff.md` — prompt guidance vs programmatic gates, money/security/compliance-always-hooks rule
- `ccaf-guide/1-agentic-architecture/1-5-agent-sdk-hooks.md` — hooks-vs-prompts deterministic/probabilistic framework
- `ccaf-learning/claude-code-in-action/automating-and-verifying-work.md`
- `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md` (pattern 2, Q53)

## stale-context-recovery (50%, 1/2)

Files:
- `ccaf-guide/1-agentic-architecture/1-7-session-state-resumption.md` — `--resume` vs `fork_session` vs fresh-start-with-summary
- `ccaf-guide/5-context-management/5-4-codebase-exploration.md` — scratchpad files, cross-phase summary injection, crash-recovery manifests
- `ccaf-learning/claude-code-in-action/long-sessions-and-steering.md`
- `exam-prep/attempts/claudecertificationguide-mockexam-try2-2026-08-18-analysis.md` (Q6, "Other misses" section)

## hook-lifecycle-events (50%, 1/2)

Files:
- `ccaf-guide/1-agentic-architecture/1-5-agent-sdk-hooks.md` — PreToolUse vs PostToolUse mechanics
- `ccaf-guide/1-agentic-architecture/1-4-workflow-enforcement-handoff.md` — SubagentStart/SubagentStop
- `ccaf-guide/3-claude-code-config/3-6-cicd-integration.md` — PreCompact (missed on try2 Q33 — assumed a nonexistent "Compact" tool hook)
- `ccaf-learning/claude-code-in-action/automating-and-verifying-work.md`

## effective-claude-md-rules (50%, 1/2)

Files:
- `ccaf-guide/3-claude-code-config/3-1-claude-md-hierarchy.md` — hierarchy, `@` imports, CLAUDE.md vs settings.json enforcement gap
- `ccaf-guide/3-claude-code-config/3-3-path-specific-rules.md` — `.claude/rules/` YAML frontmatter, glob scoping, when it beats directory CLAUDE.md
- `ccaf-learning/claude-code-in-action/long-sessions-and-steering.md`

---

## Suggested NotebookLM notebook grouping

Given NotebookLM's per-notebook source limits, group by domain rather than one giant notebook:

1. **Multi-agent orchestration** (coordinator-context-passing, role-scoped-subagent-splitting, hooks-vs-claude-md-enforcement, hook-lifecycle-events) — all of `ccaf-guide/1-agentic-architecture/`
2. **Tool design** (few-shot-vs-tool-granularity, tool-choice/tool-granularity confusion, tool description quality, prompt-keyword-overlap) — all of `ccaf-guide/2-tool-design-mcp/`
3. **Claude Code config & sessions** (worktree-coordination, stale-context-recovery, effective-claude-md-rules) — `ccaf-guide/3-claude-code-config/`, `ccaf-guide/5-context-management/5-4-codebase-exploration.md`, `long-sessions-and-steering.md`
4. **Prompt engineering** (prompt-clarity-vs-verification-patch, concrete-examples-vs-vague-prose) — `ccaf-guide/4-prompt-engineering/`

Add the two mock-exam analysis files and `.flashcards/weak-spots.md` to every notebook — they carry the "why you got it wrong" reasoning the reference pages don't have.

# 3.4 Plan Mode vs Direct Execution

## Source
https://claudecertificationguide.com/learn/3-claude-code-config/3-4-plan-mode-execution

## Summary

### Core Definitions

**Plan Mode:** a Claude Code operational state designed for complex, ambiguous tasks where exploration and strategy design must precede implementation. Plan mode "enables safe exploration and design. Claude reads the codebase, analyses dependencies, and proposes an approach — all without modifying any files."

**Direct Execution:** a mode that "skips the planning phase and makes changes straight away" for well-understood, narrowly-scoped modifications where the solution approach is already determined.

### Plan Mode: Triggering & When to Use

Plan mode activates for tasks meeting any of these criteria:

1. **Large-scale architectural changes** — "Restructuring a monolith into microservices, reorganising a module system, or refactoring a core abstraction all require understanding the existing structure before changing it."

2. **Multiple valid solution paths** — tasks where "different ways to solve the problem (e.g., different integration architectures with different infrastructure requirements)" exist, necessitating evaluation before commitment.

3. **Architectural decisions required** — decisions about "Service boundaries, module dependencies, API contracts — these decisions have downstream consequences. Planning prevents costly rework."

4. **Multi-file modifications** — "A library migration affecting 45+ files requires a consistent strategy. Without a plan, you risk applying the migration inconsistently across files."

5. **Codebase exploration necessary** — when you must "understand dependencies, trace data flows, or map the existing structure before changing anything."

### Direct Execution: When to Use

Direct execution applies when:

1. **Change is well-scoped** — "A single-file bug fix with a clear stack trace. Adding a date validation conditional. Updating a configuration value."
2. **Correct approach is known** — "You know what needs to change, where, and how. There is no design decision to make."
3. **Limited scope** — "One function, one file, one clear modification."

### Critical Distinction: Ambiguity vs. Difficulty

"The decision is not about difficulty but about ambiguity. A difficult but well-defined bug fix (clear stack trace, single function, known cause) is direct execution. A seemingly simple feature request that could be implemented three different ways and affects multiple modules is plan mode."

This directly contradicts intuitive thinking where difficulty might suggest planning is needed.

### The Explore Subagent

The Explore subagent is a specialized tool that addresses context window saturation during multi-phase tasks. It operates by:

1. "Runs the exploration in isolation."
2. "Produces summaries of its findings."
3. "Returns those summaries to the main conversation."
4. "Keeps the main context window clean for the actual implementation work."

This mechanism prevents verbose discovery output (file listings, dependency graphs, code excerpts) from degrading subsequent response quality.

### Hybrid Pattern: Plan Then Execute

A specific tested pattern combines both modes sequentially:

**Phase 1 (Plan):** "Use plan mode to explore the codebase, understand dependencies, evaluate approaches, and design the implementation strategy."

**Phase 2 (Execute):** "Switch to direct execution to implement the planned approach, file by file, with the strategy already decided."

Example given: a library migration across 30 files requires planning to "Identify all files importing the old library, map the API differences between old and new, design the migration pattern, check for edge cases," then executing "Apply the migration pattern to each file using the planned approach."

"It's plan THEN direct, not plan OR direct. The exam expects you to spot the pattern."

### Exam Traps Explicitly Called Out

**Trap 1 — Defaulting to direct execution for multi-file architectural work:** "Multi-file modifications with multiple valid approaches require plan mode. Direct execution risks costly rework when dependencies are discovered late."

**Trap 2 — Over-planning simple fixes:** "A single-function fix with a known cause and clear stack trace is the textbook case for direct execution. Plan mode adds unnecessary overhead when the problem, location, and solution are all clear."

**Trap 3 — Missing the hybrid pattern:** "The exam tests whether you know to combine plan mode for investigation with direct execution for implementation. This is the correct approach for tasks like library migrations."

**Trap 4 — Reactive complexity discovery:** "When complexity is already stated in the requirements (e.g., monolith restructuring), plan mode should be chosen upfront. The complexity is known, not speculative. Do not wait for surprises."

### Decision Framework Table

| Task Characteristic | Appropriate Mode |
|---|---|
| Architectural restructuring | Plan mode |
| Library migration (many files) | Plan mode (then direct execution) |
| Multiple valid implementation approaches | Plan mode |
| Codebase exploration needed | Plan mode (with Explore subagent) |
| Single-file bug fix with clear stack trace | Direct execution |
| Adding a validation check to one function | Direct execution |
| Configuration value update | Direct execution |
| Known fix, known location, known approach | Direct execution |

### Practice Scenario Answer

The correct answer to the practice scenario (restructure monolith, fix null pointer exception, migrate logging library) is: "Plan mode for (1) and (3), direct execution for (2)." This reinforces that architectural work and multi-file migrations require planning, while single-function fixes with clear causes do not.

## Note on worktree coverage

This page does not mention git worktrees, worktree coordination, rewind/checkpoint, or steering/interrupt mechanisms at all — it is scoped strictly to plan mode vs. direct execution and the Explore subagent. See `3-5-iterative-refinement.md` for the note on where worktree content was (not) found across this domain.

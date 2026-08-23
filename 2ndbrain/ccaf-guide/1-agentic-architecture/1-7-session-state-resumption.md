# Session State and Resumption

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-7-session-state-resumption

## Summary

### Core Concept
Session management determines how agents maintain continuity across work sessions, particularly during long-running tasks involving debugging, code review, or multi-day research where context accumulates tool results, file analyses, and reasoning chains.

### Three Session Management Options

**Option 1: `--resume <session-name>`**
Resumes a specific named session from where it left off, restoring the entire conversation history including all tool results, analyses, and reasoning.

- **When to use:** Prior context remains valid; files have not changed significantly since the last session; you want to continue exactly where work stopped.
- **When NOT to use:** Files have been modified; tool results no longer reflect current codebase state (creates the stale context problem).
- **Technical details:** Sessions are named when started with `--name` / `-n` or mid-session with `/rename`, then resumed later with `claude --resume <name>`. The `-c` / `--continue` flag resumes the most recent conversation in the current directory without explicit naming.

**Option 2: `fork_session`**
Creates an independent branch from a shared analysis baseline. Each branch operates independently with no visibility into the other branch's results.

- **When to use:** After completing initial analysis, explore divergent approaches from that shared starting point (e.g., comparing two refactoring strategies).
- **When NOT to use:** Simply continuing the same investigation; fork is for divergence, not continuation.

**Option 3: Fresh Start with Summary Injection**
Start a completely new session but inject a structured summary of prior session findings into initial context. The new session contains no stale tool results — only curated summary content.

- **When to use:** Tool results are stale (files changed, APIs updated, dependencies shifted); context degradation from long sessions with irrelevant accumulated results; when a clean baseline is needed while preserving prior knowledge.
- **When NOT to use:** Prior context is still valid and full conversation history should be maintained.

### The Stale Context Problem
**Definition:** Occurs when an agent resumes a session after code modifications and reasons from cached tool results that no longer reflect current file states.

**Manifestation example:** A developer analyzes a codebase, modifies three files, then resumes the session. Claude recommends fixes already made and references code no longer present — because old tool results remain in conversation history.

**Why it happens:** When resuming, the entire conversation history restores, including every previous tool result. If files were read during the prior session and subsequently modified, old file contents persist as tool results. The model reasons from this stale data alongside new data, creating contradictions.

**Naive fix and its limitations:** Simply resuming and asking the agent to re-read modified files. While better than nothing, stale tool results remain in history; the model may still reference old information for tangential decisions not directly involving modified files.

**Correct fix:** Start a fresh session with structured summary of prior findings. Specify which files changed so the agent performs targeted re-analysis of only those files. No stale tool results exist, and the injected summary preserves knowledge without outdated data.

### Targeted Re-Analysis vs Full Re-Exploration
**Core principle:** When files change, don't re-analyze the entire codebase. Targeted re-analysis is more efficient.

**Implementation:**
1. Start fresh session
2. Inject structured summary: "Prior analysis found X, Y, and Z across the codebase. The following 3 files have been modified since: [filenames]."
3. Agent re-reads and re-analyzes only the modified files
4. Agent combines fresh analysis of changed files with preserved summary of unchanged files

**Efficiency rationale:** Re-reading 50 files because 3 changed is wasteful; targeted approach preserves time investment.

### Decision Matrix for Scenario Selection

| Scenario | Best Option | Reasoning |
|----------|------------|-----------|
| Continuing work from yesterday, no files changed | `--resume` | Prior context valid, full history useful |
| Comparing two refactoring approaches | `fork_session` | Divergent exploration from shared baseline |
| Resuming after modifying 3 of 50 files | Fresh start + summary | Stale tool results would cause contradictions |
| Long session with cluttered history | Fresh start + summary | Degraded context benefits from clean baseline |
| Exploring testing strategy vs documentation strategy | `fork_session` | Two independent approaches from same analysis |
| Resuming after dependency updates | Fresh start + summary | Multiple files may change indirectly |

### Practical Example: The Contradictory Advice Bug
**Scenario:** Developer analyzes 50-file codebase over two days. On Day 1, analyzes authentication module, identifies three issues. Overnight, fixes all three by modifying `auth.ts`, `session.ts`, and `middleware.ts`. On Day 2, resumes session.

**Problem:** Claude recommends fixing already-fixed issues because old tool results showing unfixed code remain in conversation history. When asked about current state of `auth.ts`, Claude gives contradictory answers — sometimes referencing old code (stale tool result), sometimes referencing new code (fresh read).

**Solution:** Start fresh session with summary: "Prior analysis identified three authentication issues in auth.ts, session.ts, and middleware.ts. All three have been fixed. Please re-analyse these three files to verify the fixes and check for any new issues introduced by the changes."

**Outcome:** Fresh session has no stale tool results. Agent reads current files, verifies fixes, provides consistent advice based on actual current state.

### Exam Traps Explicitly Called Out
1. **Suggesting full re-exploration of 50-file codebase when only 3 files changed:** Full re-exploration is wasteful. Inform the agent about the specific 3 files that changed for targeted re-analysis. The prior summary covers everything else.
2. **Recommending --resume after files have been modified:** Resuming preserves stale tool results in the conversation history. The agent may reason from outdated file contents, leading to contradictory advice. A fresh start with summary injection avoids this.
3. **Confusing fork_session with --resume:** `fork_session` creates independent branches for exploring different approaches. `--resume` continues the same conversation. They serve entirely different purposes — fork for divergence, resume for continuation.
4. **Using fork_session to handle stale context after file changes:** `fork_session` branches from an existing session, which still contains stale tool results. The fork inherits the stale context. A fresh start with summary injection is the correct approach for stale data.

### Practice Scenario Analysis
**Scenario:** Developer resumes Claude Code session after modifying 3 files in a 50-file codebase. Agent gives contradictory advice about modified files — recommends changes already made and references code no longer existing.

**Correct answer:** Option C — "Start a fresh session with an injected summary of prior findings and inform the agent about the specific 3 file changes for targeted re-analysis."

**Why other options are incorrect:**
- Option A (resume + re-read): Stale results remain in history
- Option B (full fresh analysis): Wasteful re-exploration of entire codebase
- Option D (fork_session): Inherits stale context from parent session

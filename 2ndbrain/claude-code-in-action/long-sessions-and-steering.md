## Summary

**Scoping before Claude starts.** Long tasks (refactoring across a dozen files, implementing a feature) go smoother if you scope the work first rather than steer after the fact. Plan mode has Claude research in read-only mode and hand back a plan before touching anything — read the plan thoroughly, since a more thorough plan means fewer execution hiccups. Revising the plan is faster than hoping for the best: just ask Claude to change what you want changed.

**Steering mid-session.**
- `/compact` summarizes the conversation and drops the old context to free up the context window — but can lose something important, so add instructions after the command telling Claude what to keep/focus on (e.g. "focus on the API changes, drop the earlier debugging back-and-forth").
- **Rewind**: double-tap escape on an empty prompt to open the rewind menu. Every user prompt creates a checkpoint. Options: restore code, conversation, or both; **summarize from here** (compresses everything after the checkpoint — good for freeing space after a side conversation); **summarize up to here** (compresses everything before the checkpoint — good for keeping implementation detail while condensing a long setup phase).

**More autonomous control — goal and loop.**
- `/goal` sets a completion condition (describe what "done" looks like) and Claude keeps working across turns until a fast evaluator confirms it's met, instead of stopping the first time it thinks it's finished. Example: `/goal all tests and source billing pass and the type checker reports zero errors`. `/goal clear` cancels it. Constraint: the evaluator only reads the transcript, so the condition has to be checkable from Claude's own output (e.g. test results it ran) — not something it can't observe.
- `/loop` runs a prompt on an interval (fixed or self-paced) between turns — useful for polling something external like a CI run or a deploy and reacting when state changes. Escape stops it.

**Worktrees.** When multiple Claude sessions work the same codebase, worktrees give each one an independent file tree so they don't fight over files (the "two steering wheels, one car" problem). A clean worktree auto-removes on exit. A `worktree` config file at the repo root lists gitignored files to copy into each new worktree — useful for `.env` files or local config you need but don't commit.

**CLAUDE.md configuration.** CLAUDE.md is *not enforced configuration* — the longer the file, the more its rules compete with each other and the less reliably Claude follows any of them. Key points:
- If a rule is a hard, non-negotiable rule ("never push to main"), it belongs in a **pre-tool-use hook**, not CLAUDE.md — the hook can actually stop the action; CLAUDE.md can only ask.
- Four config levels, all loaded together, nothing dropped: **managed policy** (always in play), **user**, **project** (shared), and **local** (yours only, e.g. architectural decisions for your current branch that shouldn't go in the shared project file).
- Path-to-file import syntax splits a large file into smaller ones for maintainability, but imports are expanded in-line at launch — they help organization, not context size.
- Effective rules are **specific and checkable**, not vague ("follow best practices" tells Claude nothing; "put new API routes in `src/api/handlers`, one per file" does). Name the actual alternative rather than just what's disallowed ("use named exports, not default exports" beats "don't use default exports").
- Emphasis is a budget: "important"/"must" only raise a rule's priority relative to everything quieter around it — spend it on the two or three rules that actually hurt when broken.
- Treat CLAUDE.md as living, revised code: when Claude does the wrong thing, treat that as a bug report against the file and ask Claude to add the missing rule. Delete anything you can't justify, move enforcement to hooks where it belongs, and scope conventions narrowly so they only load when relevant.

## My Insights
- Reacted with "hooks are important" right after the point that hard rules belong in hooks rather than CLAUDE.md — this framing (CLAUDE.md for conventions, hooks for what can't be skipped) clicked immediately.

## Ideas
None this session.

## Challenges
None this session — this material tracked cleanly against what he already knew from CLAUDE.md usage.

## Actions
None this session.

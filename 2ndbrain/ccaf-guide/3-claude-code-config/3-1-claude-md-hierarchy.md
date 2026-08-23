# 3.1 CLAUDE.md Hierarchy, Scoping, and Modular Organisation

## Source
https://claudecertificationguide.com/learn/3-claude-code-config/3-1-claude-md-hierarchy

## Summary

### Three-Level Hierarchy

**User-level (`~/.claude/CLAUDE.md`)**
- Applies only to the individual developer.
- Located in home directory outside any repository.
- Not version-controlled; never travels through git.
- New team members cloning the repository will not receive these instructions.
- Appropriate for: personal preferences, verbosity settings, preferred output style, personal shortcuts.
- Critical exam point: instructions stored here do not propagate to teammates.

**Project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`)**
- Applies to everyone on the project.
- Located in the repository and version-controlled.
- Every developer who clones or pulls automatically receives these instructions.
- Both `.claude/CLAUDE.md` (inside the `.claude` directory) and a `CLAUDE.md` at the repository root are valid project-level locations.
- Appropriate for: naming conventions, error handling patterns, testing requirements, architecture decisions, code review checklists.
- This is where shared team standards belong.

**Directory-level (subdirectory `CLAUDE.md` files)**
- Apply when working in that specific directory.
- Used for package-specific conventions that differ from project root.
- Example: `/packages/api/CLAUDE.md` might hold REST conventions that the frontend package never needs.
- Important constraint: applies only to that directory, not across the entire project.

### Loading Order and Conflict Handling

The page explicitly corrects a common misconception about hierarchy. Quoting the official documentation: "All discovered files are concatenated into context rather than overriding each other." This means:

- Files are **concatenated**, not replaced.
- Load order is from broadest to most specific.
- User-level instructions appear in context before project-level instructions.
- Directory files load from filesystem root down to working directory.
- Within a single directory, `CLAUDE.local.md` appends after `CLAUDE.md`.

**Critical caveat on conflicts:** the documentation states: "if two rules contradict each other, Claude may pick one arbitrarily." This is not a strict precedence system. The page emphasizes: "CLAUDE.md is delivered as a user message — not as part of the system prompt — and Anthropic says 'there's no guarantee of strict compliance.'"

If a rule **must** hold consistently, the page advises: encode it in `settings.json` (which the client enforces) or in a hook (which fires at a fixed lifecycle point). The key distinction: "Settings rules are enforced by the client regardless of what Claude decides to do. CLAUDE.md instructions shape Claude's behavior but are not a hard enforcement layer."

### CLAUDE.md vs. settings.json

The page draws a critical distinction: `settings.json` has a strict precedence chain (managed policy > local > project > user, with managed always winning), while CLAUDE.md does not. The answer to "which CLAUDE.md wins on conflict?" according to the documentation is essentially "neither is guaranteed to — move to `settings.json` or a hook."

### Modular Organisation with @ Path Imports

The `@` syntax (not an `@import` keyword) enables splitting CLAUDE.md across multiple files. Exact syntax shown:

```
# .claude/CLAUDE.md

Coding standards:

@./standards/naming-conventions.md
@./standards/error-handling.md
@./standards/testing-requirements.md
```

Key behavior: referenced files are inlined at load time, exactly as if pasted directly. The page notes: "imports load eagerly." This means splitting a 600-line CLAUDE.md into six 100-line imports improves source readability but does not reduce context size Claude actually sees.

For actual context reduction, the appropriate tool is `.claude/rules/` with path-scoped frontmatter (covered in Task 3.3).

### CLAUDE.local.md

Located next to `CLAUDE.md` at any level in the hierarchy, with three distinguishing characteristics:

1. **Loading order:** appended after `CLAUDE.md` at the same level (reading last doesn't create precedence in conflicts).
2. **Git handling:** conventionally added to `.gitignore` so personal tweaks remain personal.
3. **Purpose:** for individual quirks in a specific repo — favorite scratchpad path, verbose explanation repeatedly needed, temporary debugging note.

The page characterizes it as "a project-scoped version of `~/.claude/CLAUDE.md`: same idea, narrower scope."

### The .claude/rules/ Directory

An alternative to single CLAUDE.md files, this directory holds topic-specific rule files, e.g.:
- `testing.md` — test naming, assertion patterns, fixture usage.
- `api-conventions.md` — endpoint naming, request/response schemas.
- `deployment.md` — deployment checklist, environment configuration.

Each file optionally includes YAML frontmatter with path scoping (detailed in Task 3.3). Without frontmatter, rules files load for all sessions.

### The /memory Command

Critical misconception called out: "/memory command does not load configuration files — it reveals which ones are already loaded." Configuration files load automatically; `/memory` is purely diagnostic.

Implementation note: the exam guide (v1.0) treats `/memory` as showing loaded files, but current Claude Code splits this: `/memory` lists CLAUDE.md, CLAUDE.local.md and auto-memory locations and opens them in an editor; `/context` reports what actually loaded into the session under "Memory files." For exam purposes, use the command that shows what's loaded.

### Compaction and Persistence

When `/compact` summarises a long session, project-root CLAUDE.md returns intact because Claude re-reads it from disk after compaction and re-injects it — instructions were never part of conversation history to compress.

Two things do not return automatically:
1. Nested CLAUDE.md files in subdirectories.
2. `.claude/rules/` files with `paths:` frontmatter.

Both load on demand and return when Claude reads matching files.

### The Critical Exam Scenario

The page identifies this as "the exam's favourite trap for Task Statement 3.1":

**Scenario:** Developer A (on team months) has Claude Code following team conventions perfectly. Developer B (joins team, clones repo) gets inconsistent results ignoring conventions.

**Root cause:** Conventions stored in Developer A's user-level config (`~/.claude/CLAUDE.md`) instead of project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`). User-level is not shared via git.

**Fix:** Move instructions from user-level to project-level configuration.

### Exam Traps Explicitly Called Out

**Trap 1 — New team member not receiving instructions despite same repo/branch**
- Cause: instructions in user-level config instead of project-level.
- User-level is not version-controlled or git-shared.
- Solution: move to `.claude/CLAUDE.md` for team-wide application.

**Trap 2 — Thinking /memory triggers loading**
- Reality: /memory is diagnostic, showing already-loaded files.
- Configuration loads automatically based on location hierarchy.
- /memory helps debug but doesn't activate anything.

**Trap 3 — Assuming directory-level CLAUDE.md best for cross-directory conventions**
- Reality: directory-level applies to one directory only.
- For conventions spanning multiple directories (e.g. test files throughout codebase), use path-specific rules in `.claude/rules/` with glob patterns instead.

### Practice Scenario Answer

Question: Developer A's Claude Code follows API naming conventions perfectly; Developer B (joined last week, same repo/branch) gets inconsistent naming.

Correct answer: Option C — "The API naming conventions are stored in Developer A's user-level CLAUDE.md (~/.claude/CLAUDE.md) rather than the project-level configuration."

This directly tests the core exam trap: user-level config is not git-shared, so new team members don't receive it.

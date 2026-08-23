# Flashcards results — 2026-08-21

**Scope:** Long-running sessions, steering, and CLAUDE.md configuration (targeted at weak topic `worktree-coordination`); pool had only 8 questions, all run.
**Score:** 8/8

## Q1 — Claude-md-hierarchy

**Question:** A consultancy works across 12 client projects. Each developer has personal preferences (editor keybindings), the firm has firm-wide coding standards, and each client project has its own conventions. What's the correct configuration architecture?

**Options:**
- User ~/.claude/CLAUDE.md for everything, using @ path imports to pull in project-specific files from each repo
- User ~/.claude/CLAUDE.md for personal preferences; project CLAUDE.md for firm-wide standards and client conventions; directory-level CLAUDE.md or path-scoped rules for subsystem rules (✅ correct, chosen)
- Put firm-wide standards in the user-level file so they load for every session automatically
- A single CLAUDE.md at the root of each project containing all configuration levels with clear section headings

**Result:** Correct.

**Explanation:** This maps scope to ownership: user-level for what's personal to you (not shared, not versioned), project-level for what's shared with the team via version control, and directory-level for rules that only apply to a subsystem. Cramming everything into one file or the user-level file breaks that mapping — either it stops being shareable, or it stops being scoped.

## Q2 — Claude-md-hierarchy (user-level scope)

**Question:** Why is putting shared, firm-wide standards in a user-level (~/.claude/CLAUDE.md) file the wrong choice for a multi-developer consultancy?

**Options:**
- User-level CLAUDE.md can only contain hook configuration, not conventions
- User-level CLAUDE.md always loads after project-level files, so it would be overridden anyway
- User-level CLAUDE.md applies only to that individual developer and isn't shared via version control, so a colleague cloning the project never receives it (✅ correct, chosen)
- User-level CLAUDE.md has a hard character limit too small for firm-wide standards

**Result:** Correct.

**Explanation:** It's a distribution problem, not a load-order or size problem: `~/.claude/CLAUDE.md` lives on your machine only, so it never reaches a teammate who clones the repo. Firm-wide standards need to live in the project's own CLAUDE.md, which is checked into version control and travels with the codebase.

## Q3 — Plan-mode-vs-direct-execution

**Question:** A team is debugging a complex distributed-system issue: exploring logs, tracing request flows, forming hypotheses, with no changes to the codebase yet. Partway through, they identify a one-line config fix and want to apply it immediately. What's the optimal workflow?

**Options:**
- Restrict to read-only tools for investigation, then start an entirely new session with write permissions for the fix
- Use plan mode for the entire session, including the one-line fix, to maintain consistency
- Start in plan mode for investigation, then switch to direct execution for the one-line fix once identified (✅ correct, chosen)
- Use direct execution throughout, with detailed upfront instructions describing the investigation process and potential fixes

**Result:** Correct.

**Explanation:** Plan mode fits the exploratory phase (no writes yet, need review before acting), and once the fix is concrete and small, switching to direct execution avoids ceremony that doesn't add safety at that point. Using plan mode for the whole session (including the trivial fix) or direct execution for the whole session (including risky exploration) both apply one mode to a task that has two genuinely different risk profiles.

## Q4 — Why not a new session for the fix

**Question:** Why is starting a brand-new session for a one-line fix (after investigating in plan mode within the same session) the wrong move?

**Options:**
- Plan mode can never transition to direct execution within Claude Code
- A new session would lose all the investigation context (log analysis, request tracing, hypothesis formation) that led to identifying the fix (✅ correct, chosen)
- Starting a new session always regenerates the CLAUDE.md hierarchy from scratch
- New sessions default to bypassPermissions mode, which is unsafe for a production fix

**Result:** Correct.

**Explanation:** The whole value of staying in-session is preserving the accumulated context — logs traced, hypotheses formed, the reasoning chain that led to the fix. A fresh session starts from zero and would have to rebuild all of that (or worse, apply the fix "blind" without the surrounding justification), which is exactly the cost switching sessions incurs for no safety benefit here.

## Q5 — Hard-rules-belong-in-hooks

**Question:** What's the guidance for a hard, non-negotiable rule like 'never push to main'?

**Options:**
- It should go in the project's README instead of any Claude Code configuration
- It belongs in a pre-tool-use hook, not CLAUDE.md — the hook can actually stop the action, CLAUDE.md can only ask (✅ correct, chosen)
- It doesn't matter where it lives as long as it's mentioned somewhere
- It should be repeated at the top of CLAUDE.md in all caps for emphasis

**Result:** Correct.

**Explanation:** CLAUDE.md is a request — Claude reads it and tries to comply, but nothing enforces it mechanically. A pre-tool-use hook is enforcement — it can actually intercept and block the tool call before it executes. Hard, non-negotiable rules need the mechanism that can't be talked around, not louder wording in a file Claude merely reads.

## Q6 — Effective-claude-md-rules

**Question:** Which CLAUDE.md rule is written effectively, per the guidance that rules should be specific and checkable?

**Options:**
- "Be careful with API routes"
- "Follow best practices for API design"
- "Put new API routes in src/api/handlers, one per file" (✅ correct, chosen)
- "Don't use default exports" (with no alternative named)

**Result:** Correct.

**Explanation:** This rule is checkable — you can look at a diff and verify a route landed in `src/api/handlers`, one per file, or it didn't. "Follow best practices" and "be careful" give no verifiable criterion at all, and "don't use default exports" is specific about what not to do but incomplete — it doesn't say what the alternative actually is, leaving Claude to guess.

## Q7 — Stale-context-recovery

**Question:** A Claude Code agent has been working on a feature branch for 45 minutes. Tool results from 40 minutes ago (file reads) are now stale because a colleague pushed changes to those files, and the agent is reasoning from the outdated contents. What is the best recovery strategy?

**Options:**
- Ignore the staleness since the agent will naturally prioritize newer information over older context
- Restart the same session with --continue and hope the model deprioritizes the earlier reads
- Start a fresh session seeded with a summary of the valuable findings so far, then re-read the changed files from that clean state (✅ correct, chosen)
- Stay in the current session and ask the agent to re-read the changed files — the stale results remain in context regardless

**Result:** Correct.

**Explanation:** Simply re-reading the files inside the same session doesn't erase the stale copy still sitting earlier in context — the model still has both versions and no clean signal for which one is current. A fresh session avoids that ambiguity entirely: seed it with a compact summary of the useful findings (avoiding a full context rebuild), then have it re-read from a clean slate where the only file contents present are current ones.

## Q8 — Worktree-coordination (target topic)

**Question:** Two Claude Code instances are running in separate git worktrees — Instance A extracting the payment service, Instance B extracting the inventory service — and both need to modify the same shared OrderService.java. How should the team coordinate this?

**Options:**
- Worktrees already isolate each instance's changes, so no coordination is needed for shared files
- Have both instances write to separate copies of OrderService.java and merge programmatically at the end
- Sequence it: one instance completes its edit and merges first, then the other rebases onto the updated state before making its own changes (✅ correct, chosen)
- Let both instances edit OrderService.java independently and resolve the merge conflict after both are done

**Result:** Correct. First correct answer on this topic (previously 0/2).

**Explanation:** Worktrees isolate the working directory, not the file's logical state — when two instances need to touch the same shared file, that isolation is exactly what creates the coordination problem, not what solves it. Letting both edit independently and resolving conflicts after the fact wastes work on whichever side loses the conflict resolution, and "merge programmatically" doesn't really exist for arbitrary code changes. Sequencing (one merges, the other rebases onto that before editing) is the only approach that avoids wasted or conflicting work on a shared dependency.

## Weakest topics this session

None missed this session (8/8). `worktree-coordination` moves from a 100% (0/2) all-time wrong-rate to 67% (1/3 wrong) after this session's correct answer — trending better but not yet fully confirmed as resolved. Remaining confirmed weak spot: `few-shot-vs-tool-granularity` (Building with the Claude API / Prompt Engineering Techniques), still 4/4 wrong all-time.

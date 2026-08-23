## Summary

**Worktrees isolate the working tree, not merge outcomes.** Claude Code's worktree
isolation (`--worktree`, `EnterWorktree`) enforces four checks: it blocks `Edit`/`Write`/
`NotebookEdit` targeting the main checkout, blocks Bash/PowerShell commands whose working
directory resolves to the main checkout, blocks git commands redirected into the main
checkout (`git -C`, `--git-dir`, `GIT_DIR`/`GIT_WORK_TREE`, or a `cd` into it), and blocks
shell constructs it can't statically verify stay inside the worktree. All four checks are
about **where a session is allowed to write**, not about what happens when two branches
that both touched the same file try to reunite. Nothing in worktree isolation prevents,
detects, or resolves a merge conflict — that's an ordinary git problem that starts the
moment both branches are merged back, regardless of how isolated they were while diverging.

**What worktrees actually share** (this is the root of the exam trap — "worktrees isolate
everything" is false): the repository's `.git` directory (git commands from any worktree
write to the same shared object store — `git commit` inside a worktree is a real commit
in the same repo), project-scope plugins, and saved permission approvals (an "always
allow" choice made in one worktree writes to the main checkout's
`.claude/settings.local.json` and applies everywhere). Files and branch state are the only
things actually isolated.

**The exam scenario and its trap**: two Claude Code instances, each in its own worktree,
each need to modify the *same* shared file (e.g. one extracting payment logic out of
`OrderService.java`, the other extracting inventory logic out of the same file). The wrong
answer treats worktree isolation as sufficient — "let both branches edit independently,
worktrees keep them separate, resolve any conflict at merge time." That's backwards: since
both worktrees start from the same baseline and both edit the same lines/regions of a
shared file, isolation guarantees a conflict at merge time, not prevents one. The correct
answer is to **sequence the merges**: pick an order (e.g. payment extraction merges to
main first), then have the second worktree's branch either rebase onto the
post-first-merge main, or otherwise pull in the first branch's result, *before* it
finishes its own edits to the shared file — so the second worker is editing the
already-updated file rather than a stale, independently-diverged copy. This turns a
3-way conflict (base, branch A's removal, branch B's removal) into two sequential,
low-conflict edits against a moving baseline.

**The general principle behind it**: worktrees solve file isolation; they do not solve
*work-division* planning. Before splitting work across worktrees, the coordinating human
(or coordinator agent) needs to explicitly identify any file both workers will touch and
choose one of two mitigations up front — sequence the merges (one worker's change lands
before the other starts touching that file), or partition the shared file itself (assign
each worker a distinct method/section within it) so their edits don't overlap regardless
of merge order. Neither mitigation is something worktree isolation provides automatically;
both require an explicit plan before work starts, not conflict resolution after the fact.

**Relevant mechanism for staging a sequenced merge**: `worktree.baseRef` in settings
controls what a new worktree branches from — `"fresh"` (default) branches from the
repository's remote default branch, `"head"` branches from the current local `HEAD`
(carrying unpushed commits). If worker B's worktree needs to start from worker A's
in-progress state rather than stale `main`, that's the lever — either set `baseRef: "head"`
before creating B's worktree once A has committed, or explicitly merge/rebase B's branch
onto A's branch partway through, rather than letting B branch from a `main` that hasn't
seen A's work yet.

Source: [Claude Code docs — Run parallel sessions with worktrees](https://code.claude.com/docs/en/worktrees)
(fetched 2026-08-19; official docs page has no dedicated "merge coordination" section —
this synthesis is derived from the isolation-enforcement and sharing sections plus the
missed mock-exam scenario, not a single authoritative paragraph).

## My Insights

(placeholder — review and add your own take)

## Ideas

- Worth testing hands-on: spin up two worktrees against a shared file on a throwaway repo,
  deliberately don't sequence them, and watch the actual merge conflict shape — seeing the
  3-way conflict marker for yourself would cement why "isolation ≠ conflict-free" in a way
  the abstract explanation doesn't.

## Challenges

- The official Claude Code docs page doesn't actually use the phrase "merge sequencing" or
  discuss this scenario directly — this note is a synthesis of the isolation/sharing
  mechanics plus the mock-exam's tested scenario, not a direct citation. Worth treating this
  gap-topic note as a working theory to validate against any future exam-prep source that
  covers it more explicitly, rather than as settled as the rest of the vault's course-backed
  notes.
- This is the same underlying shape as `coordinator-context-passing`: a
  parallelization/isolation mechanism (worktrees, subagent context) gets mistaken for a
  *coordination* mechanism. Isolation prevents accidental interference; it does not replace
  the explicit planning step (who owns what, in what order) that avoiding real conflicts
  still requires.

## Actions

- [ ] Review this gap-topic note and add personal insights (owner: bruno)

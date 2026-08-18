## Summary

**Skills for verification.** Skills automate procedures that get repeated — the best first one to build is a verification skill: ask Claude to refactor something, and when it finishes, a skill whose description matches ("task edited code") fires automatically. It runs the test suite, reads the diff, checks that no test was weakened just to pass, and reports pass/fail with evidence attached — so checking the work no longer depends on remembering to ask for it. The same shape fits any repeated multi-step procedure (a release checklist, a migration recipe) — if you've typed the same instructions twice, that's a skill. A skill folder can hold more than `SKILL.md`: drop a `reference.md` beside it for detail Claude only reads when it needs the depth, and scripts in the folder get *executed*, not loaded, so a skill can carry its own tooling. Keep the main `SKILL.md` lean; push heavy material into side files.

This gives a three-way division of instruction surfaces:

![[claude-md-skills-hooks.png]]

- **CLAUDE.md** — conventions that always apply.
- **Skills** — procedures/reference material tied to a specific kind of task.
- **Hooks** — rules Claude must not be able to skip, because CLAUDE.md and skills are instructions Claude *follows*, while hooks are code that *runs*.

**Permission modes.** Shift-tab cycles through them; the status bar shows which one is active:

![[permission-modes.png]]

- **manual** — read-only without prompting; everything else (edits, bash commands) asks.
- **acceptEdits** — auto-accepts file edits/reads/common filesystem operations; good for iterating on code you'll review after the fact.
- **plan** — read-only; researches and proposes without editing.
- **auto** — accepts everything, but a separate classifier model reviews each action before it runs.
- **dontAsk** — only pre-approved tools run; everything else auto-denies with no prompt. Fits CI pipelines where no one's there to approve.
- **bypassPermissions** — skips all checks (equivalent to `--dangerously-skip-permissions`); only ever run inside an isolated container or VM.

**Auto mode in depth.** The classifier guards *intent*, not correctness. It blocks actions that escalate beyond the request — production deploys/migrations, force-pushes, piping downloaded code into a shell, sending sensitive data to external endpoints, destroying session files — while allowing everyday work: local edits, installing dependencies from the lockfile, read-only requests, pushing to your own branch. Because the classifier waves through anything that isn't *dangerous* even if it's *wrong* (e.g. a broken auth refactor isn't dangerous, so it passes), auto mode should always be paired with a stop hook that runs tests — the classifier watches what Claude is trying to do, the hook confirms the code actually works. Auto mode's guardrails are still evolving, so check current docs for the block/allow lists.

**Hooks.** A hook is deterministic code at a fixed point in the agent loop — it turns "Claude usually follows this" into "Claude can't skip this." Claude Code fires ~30 hook events; the ones that matter most:
- **pre-tool-use** — fires before a tool call; the main enforcement point.
- **post-tool-use** — fires after a successful tool call; typical home for auto-formatting/linting.
- **stop** — fires when Claude wants to end its turn; can refuse if conditions aren't met.
- **subagent-stop** — same signal, for a sub-agent finishing.
- **pre-compact / post-compact** — fire around compaction. To reinject context *after* compaction, use **session-start with the compact matcher**, not post-compact.
- **instructions-loaded** — fires when a CLAUDE.md/rule file loads; useful for auditing what made it into context.
- **session-start** — primes the environment; check the source to run only on fresh starts.

For pre-tool-use, return JSON with a `permission decision` field: `allow`, `deny`, or `ask` (a 4th value, `defer`, only applies to non-interactive `-p` runs where a calling process pauses and resumes the tool later). You can also return `updated input` to modify the tool call without blocking it — e.g. redacting a secret out of a bash command — but updated input replaces the *whole* input object, so echo back any fields you aren't changing.

Exit codes also work for hooks that don't return JSON: **0** = success (stdout JSON gets parsed; plain text is ignored on most events but *is* added to context on session-start/user-prompt-submit/user-prompt-expansion, which is what makes state-preservation across compaction possible). **2** = blocking error (stderr is fed back to Claude as context; blocks almost everywhere, and on `stop` specifically means "not done yet" — note exit **1** feels like an error but does *not* block). Anything else non-zero is logged but non-blocking. Post-tool-use fires after the tool already ran, so it's too late to stop the call, though it can still feed text back. Some events (notification, session-start, file-change) ignore blocking entirely — they just show stderr and carry on.

**Verifying unsupervised runs.** Verify in proportion to how much autonomy the run had — a short supervised session needs a glance, an unattended run or CI job with nobody watching needs a real check, because no one saw it happen. Keep unattended work in auto mode rather than bypassPermissions (the classifier still reviews for danger, though never for correctness). Start from the diff itself, not Claude's summary of it: run `/code-review` to walk the changes, then look at `git diff` directly, especially files that weren't part of the original plan. The real gate is whether tests actually passed and whether Claude actually ran them (versus just claiming it did) — don't leave that to trust, wire a hook so it can't be skipped: a stop hook that runs tests and refuses to end the turn on failure, or a post-tool-use hook that lints/typechecks after every edit. A hook exiting with code 2 feeds the failure straight back to Claude, which reads and fixes it without being asked, and this fires on every run whether or not you remember to ask. A sub-agent cold-review — a fresh session/sub-agent with no memory of how the code was built, reviewing the change with no stake in the approach — also works well here and catches things the original run rationalized past.

## My Insights
- Called the CLAUDE.md / Skills / Hooks three-way split "very important" in the moment — this is the distinction he asked to save as an image for this note.
- Reacted with "What?" at the sub-agent cold-review technique for unsupervised runs — flagged as a genuinely useful idea he hadn't considered (reviewing your own unattended work with a reviewer that has zero context on how it was built).

## Ideas
None this session.

## Challenges
None this session.

## Actions
None this session.

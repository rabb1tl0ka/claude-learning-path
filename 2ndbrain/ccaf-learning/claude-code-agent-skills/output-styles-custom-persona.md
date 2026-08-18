---
date: 2026-07-25
source: https://code.claude.com/docs/en/output-styles
tags: [claude-code, output-styles, persona]
---

# Building a custom output style (e.g. business coach persona)

Purpose: change *how Claude responds* (role, tone, format), not *what Claude knows* (that's CLAUDE.md's job — see [[claude-md-load-order]]). Built for going beyond software engineering — e.g. turning Claude into a business coach, writing assistant, etc.

**Built-in styles**: Default, Proactive, Explanatory, Learning.

**Custom style file** — `~/.claude/output-styles/<name>.md` (user-level) or `.claude/output-styles/<name>.md` (project-level):

```markdown
---
name: Business Coach
description: Acts as a business coach instead of a software engineer
keep-coding-instructions: false
---

You are a business coach helping the user think through strategy, decisions, and growth.
Ask probing questions before giving advice. Frame answers in terms of goals, tradeoffs, and next actions.
```

- `keep-coding-instructions: true` → keeps Claude Code's built-in software-engineering instructions underneath (use when changing tone/format but still coding, e.g. "always lead with a diagram")
- `keep-coding-instructions: false` (default) → drops all built-in coding instructions entirely (use for a full persona swap, e.g. business coach)

**Mechanics**:
- Applies to the **main conversation only** — subagents run their own system prompt and aren't affected, except forks (which inherit the parent's full system prompt)
- Read once at session start — a change takes effect after `/clear` or a new session, not mid-conversation
- Selected via `/config` → Output style, or by setting `"outputStyle": "<name>"` directly in a settings file

See also: [[output-style-per-directory]].

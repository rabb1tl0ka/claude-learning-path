---
date: 2026-07-25
source: https://code.claude.com/docs/en/output-styles
tags: [claude-code, system-prompt, claude-md, output-styles]
---

# System prompt vs CLAUDE.md vs output styles vs --append-system-prompt

| | Persistence | Position | Editable/shared | Enforcement |
|---|---|---|---|---|
| **CLAUDE.md** | File, auto-loaded every session | User message, after system prompt | Markdown, git-versioned, team-shared | Not enforced — Claude can ignore |
| **`--append-system-prompt`** | CLI flag, must repass every invocation | Appended to system prompt | Just a string per invocation, no file/versioning | Not enforced |
| **Output style** | File (`~/.claude/output-styles/` or `.claude/output-styles/`), set via `outputStyle` in settings | Directly modifies the system prompt | Markdown w/ frontmatter, versionable | Not enforced |

Key point: only output styles and `--append-system-prompt` touch the **system prompt** itself. CLAUDE.md never does — it's always a user message.

See also: [[claude-md-load-order]], [[output-styles-custom-persona]].

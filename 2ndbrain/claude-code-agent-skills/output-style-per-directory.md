---
date: 2026-07-25
source: https://code.claude.com/docs/en/settings
tags: [claude-code, output-styles, settings]
---

# Auto-applying an output style per directory/project

Project settings (`.claude/settings.json` or `.claude/settings.local.json`) resolve to the **git repository root** (through worktrees) — one settings file covers the whole repo regardless of which subdirectory you launch `claude` from.

To make an output style automatic for a given project/vault:

```json
// .claude/settings.local.json (personal, gitignored) or .claude/settings.json (team-shared)
{
  "outputStyle": "Business Coach"
}
```

- Use `.claude/settings.local.json` for a personal preference specific to this repo (not forced on collaborators)
- Use `.claude/settings.json` if the whole team should get this style
- Precedence: managed > command line > local > project > user

Effectively gives per-directory personas: spawn Claude in one repo → business coach, spawn it in another → default coding behavior.

See also: [[output-styles-custom-persona]].

---
date: 2026-07-25
source: https://code.claude.com/docs/en/memory
tags: [claude-code, claude-md]
---

# CLAUDE.md load order across nested directories

Spawning Claude at `D/` inside `A/B/C/D/E/F/G` loads CLAUDE.md from A, B, C, D (working dir and all parents). Subdirectories below the working dir (E, F, G) are NOT loaded at launch — only on demand, when Claude reads a file in those subdirs.

**Load order: parent → child (root first, working dir last).** Within a directory, `CLAUDE.local.md` loads right after that directory's `CLAUDE.md`.

Full scope precedence (broadest → narrowest): **managed policy → user (`~/.claude/CLAUDE.md`) → project → local**.

**Conflicts**: files are concatenated, not merged/overridden. If two CLAUDE.md files contradict each other, Claude may pick arbitrarily — no guaranteed "last one wins" rule. Docs recommend periodically auditing for contradictions. For hard enforcement (not dependent on Claude's judgment), use a `PreToolUse` hook or `managed-settings.json` instead.

**Where it loads**: CLAUDE.md is delivered as a **user message after the system prompt** — not part of the system prompt itself. Regular context window content (shows in `/context`), not baked into the model's core instructions.

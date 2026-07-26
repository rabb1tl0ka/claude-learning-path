---
source: https://anthropic.skilljar.com/introduction-to-agent-skills/434525
date: 2026-07-14
---

# Introduction to Agent Skills

## Key points from the training

- **Core idea**: Skills = reusable, markdown-based instructions written once ("teach Claude once") so behavior stays consistent across sessions/workflows, instead of re-explaining the same thing every time.
- **SKILL.md structure**: frontmatter + a `description` field, which is what Claude matches against the prompt to decide relevance.
- **Progressive disclosure**: skill directories should be organized so extra detail/scripts only get pulled into context if actually needed.
- **`allowed-tools`**: skills can restrict which tools are available while that skill is active.
- **Scripts run without burning context**: code inside a skill executes but doesn't dump its logic into the context window.
- **Differentiation from other customization mechanisms**: CLAUDE.md, hooks, subagents, and skills each solve a different problem — course covers when to reach for each.
- **Distribution**: skills can be shared via repo commits, plugins, or org-wide enterprise managed settings.
- **Troubleshooting**: resolution failures, priority conflicts between skills, runtime errors.

## My insight

Skills are lazy-loaded — only the frontmatter `description` sits in context at all times; the full SKILL.md body loads only when Claude matches the prompt to that description and invokes it.

CLAUDE.md is eager-loaded, in full, every session, in this order:

1. Enterprise-managed (if set)
2. User global (`~/.claude/CLAUDE.md`)
3. Project (repo root `CLAUDE.md`)
4. Subdirectory CLAUDE.md relevant to files being touched

Practical difference: CLAUDE.md = always-on rules/context (cheap to keep short, expensive if bloated since it's paid every session). Skills = on-demand capability, cheap to have many of since only the description costs context until triggered.
</content>

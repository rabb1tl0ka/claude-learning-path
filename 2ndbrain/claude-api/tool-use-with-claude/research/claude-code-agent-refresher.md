# Defining an agent in Claude Code — one-page refresher

Standalone reference note (not a course chapter recap), requested by Bruno during the [introducing-tool-use](../introducing-tool-use.md) session as a refresher on agent syntax/structure. See also [tool-use-api-vs-claude-code.md](tool-use-api-vs-claude-code.md) for when an agent is the right call versus a tool or skill.

## File location and shape

`.claude/agents/<agent-name>.md` (project-level) or `~/.claude/agents/` for user-global agents. One file per agent. Frontmatter + a markdown body:

```markdown
---
name: agent-name
description: One or two sentences describing what this agent does and, critically, WHEN to use it — this is the field Claude matches against to decide whether to invoke it proactively.
tools: Read, Grep, Bash          # optional — restricts which tools it can use
model: sonnet                    # optional — overrides the session's default model
---

The agent's system prompt / instructions go here. Written like a briefing to a colleague who has zero context on the current conversation — explain what to do, not what the situation currently is, since the agent starts fresh with no memory of the parent conversation (unless it's specifically a "fork" type, which does inherit context).
```

## Required vs optional fields

- **`name`** — required, must match the filename (minus `.md`).
- **`description`** — required, and it's the single most important field: it is both (a) what a human sees when picking an agent by name, and (b) the signal the main loop uses to decide to invoke it *proactively* without being asked. Same failure mode as a vague tool description — too generic and it never fires or fires on the wrong turns.
- **`tools`** — optional. Omit to inherit all available tools; restrict this when the agent should only ever read, or should never touch the network, etc.
- **`model`** — optional. Omit to inherit the parent's model.

## Invocation

Two ways an agent actually runs:
1. **Explicit**: the parent calls the `Agent` tool with `subagent_type: <agent-name>` and a self-contained prompt (since, again, it has no memory of the conversation unless it's a fork).
2. **Proactive**: the main loop matches the ongoing conversation against every agent's `description` and decides to launch one itself, the same way it would decide to use a skill or a tool.

## The one thing that trips people up

An agent is not a tool call that returns inline — it's a **separate context window**. The parent never sees the agent's intermediate tool calls or reasoning, only its final text output. Write the prompt as a complete briefing (background, what's already been tried, what "done" looks like), not a terse instruction — a fresh agent can't ask a clarifying question mid-run.

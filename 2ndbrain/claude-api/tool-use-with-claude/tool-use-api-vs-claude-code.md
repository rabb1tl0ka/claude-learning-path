# Tool use (Claude API) vs. Skills and Agents (Claude Code)

Standalone reference note (not a course chapter recap), requested by Bruno during the [introducing-tool-use](introducing-tool-use.md) session — he wanted a comparison between the API's tool use and the closest equivalents in Claude Code, having guessed Skills were closest and Agents were something else.

## Claude API tool use — the raw building block

- You define a JSON Schema for each tool: `name`, `description`, `input_schema`.
- You attach the full list of tool schemas to *every* request via the `tools` parameter — the model has no memory of tools from earlier turns, you resend them each time.
- Claude decides whether to call a tool and returns a `tool_use` content block; **you** execute the actual function and send a `tool_result` block back in the next message.
- You own the entire message history, including the multi-block assistant messages tool use produces — nothing is managed for you.

This is the lowest-level primitive: maximum control, maximum bookkeeping.

## Skills — the closest analog to a "tool"

- Defined as `SKILL.md` files with frontmatter (`name`, `description`, optional `args`), auto-discovered by Claude Code from `.claude/skills/`.
- Claude decides *when* to invoke a skill the same way it decides to call an API tool: by matching the conversation against the skill's `description`. A vague description under-triggers or over-triggers, exactly like a vague tool description would.
- The key difference from raw API tool use: **you never resend the schema**. Claude Code loads the skill listing itself and keeps it available every turn — the harness handles what the API forces the developer to do by hand.

Bruno's instinct was right: Skills are the nearest equivalent to a tool definition, just with the "attach the schema to every request" step removed because the harness owns that instead of the developer.

## Agents (subagents) — a different axis entirely

- Defined as `.claude/agents/*.md`, invoked via the `Agent` tool (or auto-selected by matching their `description`, same triggering mechanism as a tool/skill).
- The distinguishing property isn't *how* they're triggered — it's that an **agent runs in its own context window** and **only its final output comes back to the caller**. The intermediate tool calls, reasoning, and message history inside the agent's run are invisible to (and not managed by) the caller.
- This maps onto exactly the case Bruno described: "there's no need or value in getting access to the intermediate steps to get the final response" — that's the situation where you reach for an agent rather than a tool/skill, regardless of API vs. Claude Code.

## One-line mapping

| Claude API concept                                              | Claude Code equivalent                       | What's abstracted away                                             |
| --------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------------------------ |
| Tool schema you resend every request                            | Skill (auto-loaded, no resend)               | Manual schema attachment                                           |
| `tool_use` / `tool_result` history you manage                   | (handled internally when a skill/agent runs) | Manual multi-block history bookkeeping                             |
| A single function call, result visible in the same conversation | Agent (Task/subagent)                        | The intermediate steps themselves — only the final answer surfaces |

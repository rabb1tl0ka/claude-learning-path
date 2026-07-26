## Source
- [Introducing tool use](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287747)
- [Project overview](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287751)
- [Tool functions](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287756)
- [Tool schemas](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287753)
- [Handling message blocks](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287757)
- [Sending tool results](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287752)

## Summary

**Why tool use exists**: Claude only has access to information from its training data by default, so it can't answer questions about real-time events on its own — e.g. "what's the weather in San Francisco right now." Tools let Claude reach outside itself to get that kind of information or take an action.

**The project for this section**: teach Claude to set reminders inside a Jupyter notebook (e.g. "set a reminder for my doctor's appointment, a week from Thursday"). This simple-looking goal exposes three real gaps in what Claude can do on its own:
- it doesn't reliably know the exact current time (only the date),
- it doesn't always get time-based addition right (e.g. 379 days from a given date),
- it has no built-in mechanism for actually setting a reminder at all.

Each gap gets solved with a dedicated tool: one to get the current time, one to add a duration to a datetime, and one to set the reminder.

**Step 1 — Tool functions**: a tool function is just a normal Python function. It gets executed automatically once Claude decides it needs the extra information the function provides, based on the function's descriptive name and argument names.

**Step 2 — Tool schemas**: each tool also needs a JSON Schema so Claude knows it exists and what arguments it takes. A complete tool spec has three parts: `name`, `description`, and `input_schema`. The description should run 3-4 sentences and cover what the tool does, when Claude should use it, and what kind of output it returns.

**Step 3 — Making the request**: the tool schemas get attached to the API call via a `tools` argument, alongside the usual `model`, `max_tokens`, and `messages`.

**Handling multi-block messages**: when Claude decides to use a tool, it returns an assistant message containing *multiple* content blocks — typically a text block (e.g. "Let me find that for you") plus a `tool_use` block carrying a tool ID, the tool's name, and its input, with `type: tool_use`. This is a step up from the plain text-only responses used earlier in the course — since Claude doesn't manage conversation history itself, the developer must preserve the entire block structure when appending to the message list, not just the text.

**Sending tool results (step 4, paused mid-lesson)**: the final step is to actually run the function Claude requested and send the result back as a `tool_result` block. The session was paused right at this point (see Challenges below) before implementing it.

## My Insights

The tool-description guidance ("3-4 sentences: what it does, when to use it, what it returns") reads almost identically to how a Skill or Agent description gets written in Claude Code — same purpose, same failure mode if it's vague. That parallel is worth keeping in mind going forward: writing a good tool schema and writing a good `SKILL.md`/agent `description` field are the same underlying skill. See [tool-use-api-vs-claude-code.md](tool-use-api-vs-claude-code.md) for the full comparison.

## Ideas

- Reimplement the three reminder tools as a custom Claude Code agent — get the current time via bash, do the date math via bash/Python, and fire the reminder via a scheduling mechanism (cron, or this environment's own scheduling tools) instead of hand-writing everything as API tool functions. See [custom-agent-for-reminders.md](custom-agent-for-reminders.md).

## Challenges

- **The core open question**: in a real server exposing this to actual users (rather than the notebook's hard-coded example messages), how do you know *which* tool schema to send with a given request? Load every tool by default just in case, or run an extra check first to figure out which tool(s) are needed before sending the full schema? The course's later chapter titles (multi-turn conversations with tools, multiple tools) didn't obviously answer this before the lesson was paused here — see [tool-loading-strategy.md](tool-loading-strategy.md) for the research done on this during the same session.

## Actions
- [x] Research whether bash+cron could back a custom Claude Code agent for the reminder tools (owner: claude) — done same session, see `custom-agent-for-reminders.md`
- [x] Compare tool use in the Claude API vs. Skills/Agents in Claude Code (owner: claude) — done same session, see `tool-use-api-vs-claude-code.md`
- [x] Write a one-page refresher on defining an agent in Claude Code (owner: claude) — done same session, see `claude-code-agent-refresher.md`
- [x] Research the optimal tool-loading strategy (load all by default vs. classify first) for an AI app server (owner: claude) — done same session, see `tool-loading-strategy.md`
- [x] Resume this chapter at "Sending tool results" for part 2 of tool use with Claude (owner: bruno) — done, see [multi-turn-conversations-with-tools.md](multi-turn-conversations-with-tools.md) onward

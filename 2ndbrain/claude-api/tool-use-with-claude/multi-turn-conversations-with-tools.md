## Source
- [Multi-turn conversations with tools](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287750)

## Summary

**Why a single tool call often isn't enough**: a user query can require *multiple* tools used in sequence. Example from the course: "what day is 103 days from today?" — Claude first needs `get_current_date_time` to find today's date, then `add_duration_to_date_time` to add 103 days to it. Behind the scenes: Claude responds with a `tool_use` block for the first tool, you call it and send the result back, Claude realizes it still doesn't have enough information and responds with a *second* `tool_use` block, you call that one too and send its result back — only then does Claude have enough to answer.

**The design implication**: since real users can ask anything, you can't predict how many tool calls a given query will need. Every application that wires up tool use has to assume Claude *might* want to chain several tool calls in a row, and structure the code as a loop: submit → check the response for a `tool_use` block → if present, run it and loop again → if absent, that's the final answer to deliver.

**Detecting whether Claude wants a tool — the `stop_reason` field**: rather than manually scanning the response for a `tool_use` content block, check `stop_reason` on the response message. Possible values:
- `tool_use` — Claude wants to call a tool (the one to check for most often)
- `end_turn` — Claude finished its answer
- `max_tokens` — hit the token limit
- `stop_sequence` — hit a provided stop sequence

## My Insights

Bruno tied this straight back to the tool-loading-strategy question from the previous session: instead of sending every tool's full schema on every request (what the course does), you could expose just the tool *names* up front plus one additional lookup tool that Claude can call to fetch a specific tool's full schema on demand — only when it's actually decided it wants to use that tool. He expected this pattern to show up here, since "multi-turn conversations with tools" felt like the natural place for it, but the course doesn't reference it at all.

## Ideas

(none new this session — see the lookup-tool idea from [introducing-tool-use.md](introducing-tool-use.md) / [tool-loading-strategy.md](tool-loading-strategy.md), which this chapter's material didn't extend)

## Challenges

- If Claude only gets a tool's full schema after it "decides" it wants to use it, how does it know a tool like `get_current_date_time` exists at all, or that it should reach for it, before ever seeing that schema? The course's example doesn't address this — see [tool-loading-strategy.md](tool-loading-strategy.md) for the research already done on this exact question in the previous session.

## Actions
- [x] Check why the course page for this chapter never explains how Claude knows about a tool like `get_current_date_time` before it has the schema (owner: claude) — resolved by pointing back to [tool-loading-strategy.md](tool-loading-strategy.md) from the prior session, which already covers this; no new research needed.

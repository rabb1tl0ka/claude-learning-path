Research note, requested by Bruno during the "Using multiple tools" session: when Claude returns more than one `tool_use` block in a single response, are there ever dependencies between them (e.g. "run this one first, feed its result into the next")?

## Findings

- The API itself doesn't prescribe an execution order for multiple `tool_use` blocks in one response — you can run them concurrently, sequentially in the order they appear, or however fits your tools.
- The API has no built-in concept of one tool call depending on another's output within the same batch. If Claude genuinely needs tool B's result to call tool C, it won't emit both in the same response — it'll wait for B's `tool_result` and then request C in the next turn. Multiple `tool_use` blocks in a single response are only ever independent calls Claude judged safe to fire together (e.g. "get the weather in Paris" and "get the weather in Tokyo").
- What you must respect is **result ordering**, not execution ordering: whatever order Claude emits the `tool_use` blocks in, your `tool_result` blocks (matched by `tool_use_id`) must come back in that same order in the next user message, or the request fails with an HTTP 400.
- If your own tools do have real side-effect dependencies (e.g. one writes a file the other reads), that's your call to make — you decide whether to run them sequentially and stop on first failure, or run in parallel and return `is_error: true` for any call whose prerequisite hadn't finished (Claude will just reissue it next turn).
- You can nudge Claude's batching behavior via the system prompt — e.g. "Only batch tool calls that are independent of each other" — if you find it grouping calls that shouldn't run together.

Bottom line: dependency handling isn't an API feature, it's application logic. Claude already avoids requesting genuinely-dependent tools in the same turn.

Sources:
- [Parallel tool use](https://platform.claude.com/docs/en/agents-and-tools/tool-use/parallel-tool-use)
- [Handle tool calls](https://platform.claude.com/docs/en/agents-and-tools/tool-use/handle-tool-calls)
- [How tool use works](https://platform.claude.com/docs/en/agents-and-tools/tool-use/how-tool-use-works)

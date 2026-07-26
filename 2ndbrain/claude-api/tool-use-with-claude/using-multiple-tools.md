## Source
- [Using multiple tools](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287749)

## Summary

Wiring the two remaining project tools — `add_duration_to_date_time` and `set_reminder` — into the system already built. Both function implementations and their JSON schemas were pre-written and just needed connecting:
- `set_reminder` doesn't actually schedule anything for real — it just prints a statement like "set a reminder at this time with this content."
- Both schemas get added to the `run_conversation` function's list of tools passed to Claude.
- `run_tool` (the function that receives a tool name + arguments and calls the matching implementation) gets an additional `elif` branch per new tool.

With that, the project now has all three tools — `get_current_date_time`, `add_duration_to_date_time`, `set_reminder` — wired end-to-end.

## My Insights

Bruno called out that the course's approach here — sending *all* tool schemas to Claude on every request — is "the simplest thing they can do," not the best approach. He reiterated that the better pattern is exposing just tool *names* up front plus a single lookup tool that fetches full schemas on demand for whichever tool Claude indicates it wants (see [tool-loading-strategy.md](tool-loading-strategy.md)). He's tracking this as a known simplification in the course material rather than something to imitate directly in his own implementations.

Separately, when the transcript covered handling multiple tool calls returned in one response, Bruno wanted to know if there's ever a real dependency between them (e.g. "run this one first, then this one"). See [tool-call-dependencies-and-ordering.md](tool-call-dependencies-and-ordering.md) — short answer: the API has no such concept, Claude simply won't emit dependent calls together in the first place.

## Ideas

(none new — see the lookup-tool idea already tracked in [introducing-tool-use.md](introducing-tool-use.md))

## Challenges

(none new this lesson)

## Actions
- [x] Research whether the Claude API has a notion of dependency/ordering between multiple `tool_use` blocks in a single response (owner: claude) — done same session, see [tool-call-dependencies-and-ordering.md](tool-call-dependencies-and-ordering.md)

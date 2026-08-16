## Source
- [The web search tool](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287755)

## Summary

Another built-in tool, but with a crucial difference from the text editor tool: **web search is fully implemented by Claude/Anthropic** — no server-side implementation to write at all. Claude decides when to search (e.g. for current events), runs the search itself, and uses the results to answer.

To enable it, you add a small schema to your `tools` list with:
- `type`: `web_search_20250305`
- `name`: `web_search`
- `max_uses`: caps how many searches Claude can run per request (5 was the suggested default)

A single search can return multiple results, and depending on what those results contain, Claude may decide to run a follow-up search.

## My Insights

Bruno's reaction to the schema itself: the `type` string (`web_search_20250305`) reads like an arbitrary/date-stamped value, while `name` (`web_search`) is the one that actually has to match — he found it "crazy" that the course doesn't explain this distinction or where that type string comes from. (It's Anthropic's versioned identifier for that built-in tool definition — same general pattern as the text editor tool's `text_editor_20250728` type string.)

His bigger question was *why* web search gets a free, automatic implementation while the text editor tool requires you to write one yourself, when the course itself doesn't explain the distinction. See [server-tools-vs-client-tools.md](research/server-tools-vs-client-tools.md) — short version: web search never needs to touch anything private to you, so Anthropic can safely run it entirely on their own infrastructure; file edits and bash commands are meaningless without access to *your* environment, so those stay client-side no matter how "built-in" their schema is.

## Ideas

(none new this lesson)

## Challenges

(none new — resolved via the research note above)

## Actions
- [x] Explain why the web search tool has an automatic implementation while the text editor tool requires a manual one (owner: claude) — done same session, see [server-tools-vs-client-tools.md](research/server-tools-vs-client-tools.md)

## Source
- [Quiz on tool use with Claude](https://anthropic.skilljar.com/claude-with-the-anthropic-api/289122)

## Summary

Seven-question review quiz closing out the "Tool use with Claude" section. All seven answered correctly:
1. How to tell if Claude wants another tool call → check `stop_reason` for `tool_use`
2. Message structure when Claude uses a tool → multi-block messages with both text and `tool_use` blocks
3. Main purpose of a JSON schema for a tool → tells Claude what arguments the function expects and how to use it
4. What problem the "batch tool" solves → reduces back-and-forth communication when multiple tools are needed (see caveat below)
5. Correct sequence of the tool use workflow → initial request → tool request → data retrieval → final response
6. What lets Claude get current, real-time information by default → tools
7. What differentiates the built-in text editor and web search tools → both provide a schema, but you may still need to implement some of the functionality yourself (this is truer of the text editor than web search)

## My Insights

Bruno flagged question 4 in real time — he answered it correctly from memory but wasn't confident the course had actually taught "the batch tool" anywhere in this section, and asked Claude to track it down. It hadn't: there's no literal "batch tool" in the tool-use material. See [batch-tool-vs-message-batches-api.md](research/batch-tool-vs-message-batches-api.md) — the quiz is referencing the separate Message Batches API (bulk async processing of many independent requests), which is a different mechanism from reducing tool round-trips within one conversation. Worth noting as a slightly misleading quiz question rather than a comprehension gap.

## Ideas

(none new this lesson)

## Challenges

(none new — resolved via the research note above)

## Actions
- [x] Summarize the "batch tool" referenced in the quiz and its purpose (owner: claude) — done same session, see [batch-tool-vs-message-batches-api.md](research/batch-tool-vs-message-batches-api.md); turns out it's a mislabeling of the Message Batches API

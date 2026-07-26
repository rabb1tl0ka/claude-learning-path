## Source
- [Response streaming](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287734)

## Summary
Problem: a full non-streamed response can take 10-30 seconds depending on input/output size. Showing a spinner the whole time is a poor UX — users expect near-immediate feedback.

**Streaming** fixes this: the server still sends one user message to Claude, but Claude immediately acknowledges (empty initial response, no content yet) and then sends a stream of events containing chunks of the generated text as they're produced. The server forwards each chunk to the client as it arrives, so text appears piece-by-piece instead of all at once.

**Stream event types:**
- `message_start` — new message beginning
- `content_block_start` — a new content block begins
- `content_block_delta` — the actual text chunks
- `content_block_stop` — current block finished
- `message_delta` — message is complete
- `message_stop` — final event, no more content

## My Insights
Recapped the event types in my own words during the video as a self-check — landed on the same list as the material, good sign the mental model is holding.

## Ideas

## Challenges

## Actions

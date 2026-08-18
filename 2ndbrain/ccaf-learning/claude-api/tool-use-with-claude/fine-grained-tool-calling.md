## Source
- [Fine grained tool calling](https://anthropic.skilljar.com/claude-with-the-anthropic-api/313160)

## Summary

**Streaming + tools introduces a new event type**: `input_json`. It carries two properties — `partial_json` (a fragment of the tool call's arguments as they're generated) and `snapshot` (the cumulative JSON assembled so far). A `chat_stream` function opens the response stream and processes chunks as they arrive, branching on event type.

**Why tool-argument streaming feels laggy by default**: the Anthropic API buffers generated JSON and validates it against the tool's schema before releasing it to you — but only a *top-level key* at a time, not the whole object. E.g. for a schema with top-level keys `abstract` and `meta`, the API waits until the closing quote of the `abstract` value is generated, validates just that key/value pair against the schema, and only then releases all the individual chunks that made it up (they arrive at your server almost simultaneously because they'd been buffered). It then repeats the same wait-validate-release cycle for `meta`. The result: long pauses followed by a sudden burst of "streamed" text, rather than smooth chunk-by-chunk delivery.

**Fine-grained tool calling turns this off**: passing `fine_grained=True` disables the API-side JSON validation step entirely. Claude's generated chunks get joined into small groups and streamed down as they're produced, giving a much more traditional streaming feel for tool arguments. The trade-off: since validation is disabled, your own code must now assume it might receive **invalid JSON** and handle that gracefully — the API is no longer guaranteeing well-formed output before it reaches you.

## My Insights

(none flagged this lesson — mechanics-only)

## Ideas

(none new)

## Challenges

(none new)

## Actions

(none new this lesson)

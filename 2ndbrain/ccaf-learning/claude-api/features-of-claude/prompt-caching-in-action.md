## Source
- [Prompt caching in action](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287774)

## Summary

- Implemented in the "003 caching" notebook: a chat function updated to always cache tool schemas and the system prompt by default.
- Best practice for caching tool schemas: clone the tools list, clone-and-modify the last tool schema to add `cache_control: {"type": "ephemeral"}`, then assign the cloned list — avoids mutating the original tools list (relevant if the tool order/list changes elsewhere in the app, which could otherwise create unintended extra breakpoints).
- System prompts are cached similarly: replace the plain string with a list containing a text block (longhand form) plus `cache_control`.
- Verified in practice: a request with no tools/system prompt shows a plain token usage count; adding the (large, ~1.7K-token) tool schemas produces `cache_creation_input_tokens` on the first call, then `cache_read` tokens on an identical follow-up. Changing anything in the tools (even a single-letter schema edit) invalidates the cache and forces a new cache write.

## My Insights

- None this session — mostly followed along with the implementation.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.

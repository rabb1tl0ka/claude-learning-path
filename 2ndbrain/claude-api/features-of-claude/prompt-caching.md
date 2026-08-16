## Source
- [Prompt caching](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287772)

## Summary

- The problem: on a normal request, Claude does a large amount of internal processing on the input message before generating output — then throws all of that work away once the response is sent. A follow-up request that repeats the same earlier messages forces Claude to redo that same processing from scratch.
- Prompt caching solves this by saving that initial processing work into a temporary store. A follow-up request with **identical** earlier content can then reuse the cached work instead of reprocessing it, speeding up generation and reducing cost.
- This directly addresses latency/cost in multi-turn conversations, where the growing history is otherwise reprocessed in full on every turn.

## My Insights

- Correctly predicted what the "initial work" Claude discards actually consists of — tokenizing the prompt, generating embeddings for each token, adding context from surrounding text, then generating the output — before the video confirmed it.
- Wondered what real use cases actually hit the "same content, high frequency" sweet spot for caching to matter.

## Ideas

- None this session.

## Challenges

- What concrete examples fit the "same content sent extremely frequently" caching use case.

## Actions

- [x] Give examples of use cases where prompt caching's high-frequency benefit really applies (owner: claude) (done same session, see `research/prompt-caching-high-frequency-examples.md`)

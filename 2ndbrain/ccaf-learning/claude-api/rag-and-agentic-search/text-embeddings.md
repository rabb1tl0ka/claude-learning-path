## Source
- [Text embeddings](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287759)

## Summary

- After chunking, the next step in a RAG pipeline is waiting for a user query, then finding which chunks relate to it — fundamentally a search problem. The most common approach is **semantic search**, using text embeddings.
- A text embedding is a numerical representation of a text's meaning, produced by an embedding model: feed in text, get back a long list of numbers (roughly ranging from -1 to 1).
- It's helpful to imagine each number as a "score" for some quality (e.g. "how happy," "how much about fruit"), but in reality we don't know what any individual number actually represents — that's just a useful mental model, not a literal one.
- Anthropic doesn't provide embedding generation itself; the course uses **Voyage AI** instead (separate account/API key, stored as `voyage_api_key` in the env file), with a `generate_embedding` helper function provided in the "002 embeddings" notebook.

## My Insights

- Skeptical of relying on embedding scores without knowing what they represent — specifically questioned whether the sign (positive vs. negative) of a score gives any directional information at all.

## Ideas

- None this session.

## Challenges

- How to actually interpret an embedding's individual numeric scores — pushed back on "just trust it works."

## Actions

- [x] Explain what individual embedding score values represent, and whether the sign carries directional meaning (owner: claude) (done same session, see `research/embedding-score-interpretation.md`)

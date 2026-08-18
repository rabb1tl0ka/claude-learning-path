## Source
- [Implementing the RAG flow](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287761)

## Summary

- Walked through the "003 vector db" notebook, implementing the 5-step flow for real using a custom `VectorIndex` class.
- Step 1: chunk the report (`report.md`) using the existing `chunk_by_section` function.
- Step 2: generate an embedding for every chunk via `generate_embedding` — now updated to accept either a single string or a list of strings, returning one embedding per string when given a list.
- Step 3: create a `VectorIndex` (vector store) instance, loop over `zip(embeddings, chunks)`, and call `store.add_vector(embedding, {"content": chunk})` for each pair.
- Key point: the embedding alone isn't useful to a developer — raw numbers have no human-readable meaning on their own, so you must store the original text (or at least an ID pointing back to it) alongside the embedding when inserting into the vector database.

## My Insights

- None this session — mostly restating/note-taking during this portion.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.

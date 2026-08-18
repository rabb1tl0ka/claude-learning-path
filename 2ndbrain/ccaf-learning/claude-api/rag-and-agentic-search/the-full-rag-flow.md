## Source
- [The full RAG flow](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287764)

## Summary

- Merges the three topics covered so far (RAG overview, text chunking, text embeddings) into one complete step-by-step pipeline, illustrated with a toy 2-dimensional embedding model for clarity.
- The five steps:
  1. Chunk the source document into separate pieces of text.
  2. Generate an embedding for each chunk.
  3. Normalize each vector to a magnitude of one (handled automatically by the embedding API — no manual math needed).
  4. Store the normalized vectors in a specialized vector database, optimized for storing/comparing/searching long lists of numbers.
  5. At query time, embed the user's question and use **cosine similarity** (the cosine of the angle between two vectors) to find the most similar/relevant chunks.
- Worked example uses a hypothetical embedding model that always returns vectors of length two, with each dimension explicitly pre-labeled (e.g. "how much about medical," "how much about software engineering") purely to build intuition — real embedding models don't expose interpretable axes like this.

## My Insights

- None this session — commentary here was mostly Bruno reading the slide content aloud rather than adding his own take.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.

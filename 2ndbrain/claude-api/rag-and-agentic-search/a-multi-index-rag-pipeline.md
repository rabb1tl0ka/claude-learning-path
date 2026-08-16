## Source
- [A Multi-Index RAG pipeline](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287766)

## Summary

- To combine semantic (vector) search and lexical (BM25) search, wraps both index implementations — which already expose near-identical public APIs (e.g. `add_document`, `search`) — inside a new `Retriever` class.
- The retriever forwards a user's question to both the vector index and the BM25 index, then merges their two result lists.
- Merging uses **Reciprocal Rank Fusion (RRF)**: worked example — a vector search returns chunks `[2, 7, 6]` and BM25 returns `[6, 2, ...]`. Each chunk gets a score of `1 / (1 + rank)` per list it appears in; the per-list scores are summed, and chunks are re-sorted by this combined score from greatest to least.
- In the worked example, this produced section 2 as most relevant, then section 6, then section 7 — RRF prioritizes rank *position* across the two lists, not the raw underlying similarity/BM25 score.

## My Insights

- Found it interesting that RRF only considers rank position and ignores the raw underlying score entirely — reasoned through it himself and concluded that's fine, since the score is what determined the rank in the first place.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session — his closing line ("next we're going to go for the next chapter called Features of Claude") was just course navigation, not a commitment.

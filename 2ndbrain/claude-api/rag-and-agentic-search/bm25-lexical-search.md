## Source
- [BM25 lexical search](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287767)

## Summary

- Demonstrated a real failure case for semantic search: searching "what happened with incident 2023 Q4011" correctly surfaces Section 10 (which is genuinely about the incident) first, but the second result is Section 3 (financial analysis), which never mentions the incident at all — a surprising wrong result from an otherwise solid technique.
- To address this, introduces lexical search via **BM25** as a parallel technique alongside semantic search.
- BM25 works like classic keyword search: breaks the query into individual words, gives higher weight to rare/specific terms, ignores common words, and focuses on term frequency rather than meaning — making it excel at exact technical identifiers and specific phrases that semantic search can miss.
- Semantic and lexical search are complementary: semantic search understands context and meaning, while lexical search guarantees you don't miss exact term matches. Combining both creates a more robust retrieval system that handles both conceptual queries and precise lookups.

## My Insights

- Reacted with surprise ("what is this about, bro?") to seeing the semantic-search failure case play out live in the notebook.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.

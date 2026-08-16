## Source
- [Text chunking strategies](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287776)

## Summary

- Worked "bug" example: a report split per-line, where a medical-research line contains the word "bug" and a software-engineering line contains "infection vectors." A naive match on "how many bugs did engineers fix this year?" pulls the wrong (medical) chunk purely on keyword overlap — illustrating how chunking + retrieval together can badly mislead.
- Three chunking strategies:
  1. **Size-based** — equal-length string splits (e.g. a 325-char document → 3 chunks of ~108 chars). Simplest to implement, most common in production; risks cutting words/sentences mid-way and losing context like a section header. An **overlap** variant includes some characters from neighboring chunks to preserve context, at the cost of duplicated text across stored chunks.
  2. **Structure-based** — split on document structure (headers, paragraphs, sections), e.g. Markdown's `##` headers. Easy when the format guarantees this structure, but brittle for plain text or PDFs without consistent formatting.
  3. **Semantic-based** — use NLP to group related sentences/sections together. More advanced technique, not implemented in this course.
- No single "correct" chunking strategy — the right choice depends on document structure and use case.

## My Insights

- Surprised the chunk search in the "bug" example seems to stop at the first matching chunk — expected multiple chunks to surface, with the correct (software) chunk appearing alongside the wrong (medical) one if more bug-related chunks existed elsewhere in the document.
- Skeptical of size-based chunking as a strategy at all — reasoned that a purely static split will inherently produce incomplete sentences, and questioned how that could ever be considered good.

## Ideas

- None this session.

## Challenges

- Wants to understand how the retrieval step actually decides what counts as "relevant," and whether it can/should return multiple chunks rather than stopping at the first hit.

## Actions

- [ ] Resume the video from text chunking strategies into part two — session was paused right after covering size/structure/semantic chunking (owner: bruno)

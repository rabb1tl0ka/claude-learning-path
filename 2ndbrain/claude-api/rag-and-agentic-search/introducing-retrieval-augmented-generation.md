## Source
- [Introducing Retrieval Augmented Generation](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287763)

## Summary

- Core problem RAG solves: answering specific questions against very large documents (100–1,000 pages) without dumping the whole thing into the prompt.
- **Option 1 (direct prompting)** — stuff the full document text into the prompt. Breaks down because Claude has hard input limits, quality degrades as prompts grow longer, and cost/latency scale with prompt size.
- **Option 2 (RAG)** — chunk the document ahead of time; at query time, find the chunk(s) most relevant to the question and insert only those into the prompt.
- Upsides: focuses Claude's attention on relevant content, scales to huge or multiple documents, smaller prompts mean faster and cheaper responses.
- Downsides: real preprocessing complexity (chunk the document, build a search mechanism, define "relevant"), and no guarantee a retrieved chunk carries all the context needed — e.g. risk factors could be split across a dedicated risk section and a separate strategy/outlook section elsewhere in the document.
- Many possible ways to chunk a document — covered in the next chapter (text chunking strategies).

## My Insights

- Half-joked he could've gotten the same understanding from reading the page instead of watching the video — for slide-heavy/conceptual chapters, reading may be faster than watching.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.

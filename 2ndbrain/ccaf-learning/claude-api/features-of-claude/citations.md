## Source
- [Citations](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287771)

## Summary

- The problem citations solve: when Claude answers using PDF/document content, a user might assume Claude is answering purely "from memory" rather than grounded in a specific source — citations make that sourcing explicit.
- Enabled by adding a `citations` field (`enabled: true`) to the source block alongside the PDF/document.
- With citations on, the response's content list includes text blocks paired with a **citation page location**: cited text (the actual source text supporting the statement), document index, document title, and start/end page.
- Intended use: build a UI that shows exactly where in the source document each part of Claude's answer came from, rather than presenting the answer as if generated from nowhere.

## My Insights

- Affirmed citations as important for grounding answers in the actual source data used.

## Ideas

- None this session.

## Challenges

- None this session.

## Actions

- None this session.

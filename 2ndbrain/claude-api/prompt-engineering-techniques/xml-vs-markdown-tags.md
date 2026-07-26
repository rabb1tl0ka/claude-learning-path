# XML tags vs Markdown-style tags for prompt structure

Standalone reference note (not a course chapter recap), requested by Bruno during
the [prompt-engineering-techniques](prompt-engineering-techniques.md) session — he
wondered whether wrapping content in XML tags (`<sales_records>...</sales_records>`)
has any real advantage over the Markdown-bracket style he already uses in his own
second-brain notes (`[sales_records] ... [/sales_records]`).

## Short answer

Use XML tags for anything you'd call a "real prompt" (production pipelines,
multi-component prompts, API calls). Markdown-style brackets are fine for personal
notes, but they're not an equivalent substitute inside a prompt sent to Claude.

## Why XML wins for prompts specifically

- **Training bias**: Claude was fine-tuned on a large volume of XML-tagged data,
  so it parses `<tag>...</tag>` boundaries more reliably than ad-hoc bracket
  conventions it wasn't specifically trained to recognize as delimiters.
- **Unambiguous boundaries**: `<tag>` / `</tag>` pairs are a fixed, well-known
  pattern. A homemade `[tag]` / `[/tag]` convention can visually collide with
  other bracket usage in the same prompt (arrays, citations, markdown links
  `[text](url)`), which XML angle brackets don't.
- **Anthropic's own docs recommend it explicitly** as the primary structuring
  mechanism for complex prompts — official guidance reports meaningfully more
  consistent outputs on structured vs. unstructured equivalents.

## XML and Markdown aren't actually competing

They operate at different levels and combine well:

- **XML tags** structure the *semantic role* of a prompt section (`<context>`,
  `<instructions>`, `<examples>`, `<sales_records>`).
- **Markdown** (headers, bullets, bold) structures the *content inside* a
  section — e.g. a bulleted list inside `<instructions>...</instructions>`.

So the pattern isn't "XML vs Markdown," it's "XML for the outer skeleton,
Markdown for formatting inside a tag" when both are useful.

## Practical guidance

- Wrap each distinct type of content in its own tag (`<instructions>`,
  `<context>`, `<input>`) — one tag type per kind of content reduces
  misinterpretation.
- Order matters less than consistency, but a common convention is
  `<role>`/`<context>` first, then `<task>`, then `<instructions>` /
  `<output_format>`.
- Nest tags for hierarchical content: `<outer><inner>...</inner></outer>`.
- Skip XML entirely for simple, single-instruction conversational prompts —
  it adds token overhead and complexity that isn't worth it below a certain
  prompt complexity.

## For his own second-brain notes specifically

The bracket convention (`[tag]`/`[/tag]`) Bruno uses in his notes is fine to
keep — that's plain personal markdown, not a prompt sent to a model, so
Claude's XML training bias doesn't apply there. The two conventions only
actually compete when the content in question is going into a Claude prompt.

Sources:
- [Prompting best practices - Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-prompting-best-practices)
- [Use XML tags to structure your prompts - Anthropic docs](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/use-xml-tags)

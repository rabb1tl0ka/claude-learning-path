## Source
- [Prompt engineering](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287745)
- [Being clear and direct](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287744)
- [Being specific](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287740)
- [Structure with XML tags](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287741)
- [Providing examples](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287746)
- [Exercise on prompting](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287748)
- [Quiz on prompt engineering techniques](https://anthropic.skilljar.com/claude-with-the-anthropic-api/289121)

## Summary
Prompt engineering = taking a written prompt and improving it iteratively to get more reliable, higher-quality output. The module's workflow: set a goal → write an initial (deliberately weak) prompt → evaluate it with the eval pipeline from the previous module (starting grade: 2.32) → apply a technique → re-evaluate, repeating per technique.

**Being clear and direct** — the first line of a prompt matters most. Clear = simple language. Direct = command-style instructions with action verbs ("write," "create," "generate") instead of questions, giving Claude both an action and a specific task up front.

**Being specific** — add guidelines or steps to constrain Claude's output space (e.g. an unconstrained "write a story" prompt can vary wildly in length/characters). Two guideline types: (a) quality/attribute lists (length, structure, required elements) and (b) step-by-step instructions that direct the model's process.

**Structure with XML tags** — when a prompt interpolates a lot of content (e.g. 20 pages of sales records), wrapping it in a descriptive XML tag (`<sales_records>`) helps Claude parse what each chunk of text represents. Tag names are arbitrary but should be descriptive.

**Providing examples** — one of the most effective techniques. One-shot = single example; multi-shot = multiple examples, useful for covering edge cases.

**Quiz**: 5/5.

## My Insights
- Restated the clear/direct rule in his own words: "Clear means simple language anyone can understand. Direct means instructions, not questions — start with direct action verbs like write, create, generate."
- Flagged (and then self-corrected) a broader observation: if building an AI product, the system prompt and prompt types embedded into the system are "the uniqueness about it" — called it more of a highlight than a fully-formed idea.
- Wondered whether his own second-brain habit of using Markdown-style bracket tags (`[tag]`/`[/tag]`) instead of formal XML tags makes a difference — this became the Claude action below. See also: [`xml-vs-markdown-tags.md`](xml-vs-markdown-tags.md).

## Ideas
*(none — the AI-product observation above was self-classified as an insight, not an idea)*

## Challenges
*(none flagged — skipped the hands-on exercise and went straight to the quiz, no comprehension gaps noted)*

## Actions
- [x] Research XML tags vs Markdown tags for structuring prompts (owner: claude) — done same session, see [`xml-vs-markdown-tags.md`](xml-vs-markdown-tags.md)

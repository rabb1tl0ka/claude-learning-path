## Summary

When an extraction pipeline (or any research/synthesis system) needs to preserve where a piece of information came from, the intuitive approach is inline prose citation: "According to page 12 of the report, revenue grew 14%." This looks fine as a standalone sentence, but it's fragile the moment that text gets processed further — which in a pipeline, it almost always does. 

If a downstream step summarizes, paraphrases, re-formats, merges with other sources, or truncates the text (all common: another LLM call, a template renderer, a length-limited UI card), the citation is just more prose sitting next to the fact. There's nothing structurally binding the citation to the specific claim, so it's exactly the kind of detail that gets silently dropped or detached during summarization/rewriting. The framing that shows up in this space: "attribution dies during summarisation" unless it's explicitly preserved as structured data rather than prose.

The robust alternative: emit **structured claim-source pairs** instead of embedding the citation in prose. Each individual finding/fact becomes its own record with separate fields, typically something like: ```json { "claim": "Revenue grew 14% year-over-year", "source_url": "...", "document_name": "Q3 2026 report", "excerpt": "...", "page": 12, "date": "2026-07-01" } ``` Because the source metadata (URL/document name/page/date) lives in its own fields rather than inline in the claim's text, it survives any downstream transformation that touches the `claim` field — summarization, re-templating, merging with other claims — without needing a separate lookup database or ID-matching scheme to reconnect a claim back to where it came from. The source travels *with* the claim as data, not as text that has to be parsed back out.

This also naturally handles conflicting sources: rather than picking one value when two sources disagree, keep both claim-source records intact (each with its own date/source) so a downstream consumer can weigh recency or authority themselves, instead of the pipeline silently discarding one.

## My Insights

Ties to citations as an actual Claude API feature covered in `claude-api/features-of-claude/citations.md` — that feature already returns structured citation objects (mapping generated text back to source document locations) rather than inline prose, so this gap-topic is really the same design principle Anthropic's own citations API already encodes, generalized to any multi-step pipeline that isn't necessarily using that specific API feature.

## Challenges

- Structured claim-source pairs add real overhead (schema design, more fields to populate and validate per claim) compared to just writing a sentence with a citation baked in — worth it once anything downstream will reprocess the text, probably not worth it for genuinely single-shot, human-read-once outputs.
- Merging/deduplicating claims that say the same thing from multiple sources without losing any individual source's attribution is non-trivial once claim volume is high.

## Actions

- [ ] Review this gap-topic note and add personal insights (owner: bruno)

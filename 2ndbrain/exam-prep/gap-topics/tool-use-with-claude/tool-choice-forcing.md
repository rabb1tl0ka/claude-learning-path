## Summary

The Claude API's `tool_choice` parameter controls whether/how Claude is required to call a tool on a given request, separately from which tools are made available. It takes a few forms:

- **`auto`** (the default): Claude decides for itself whether calling a tool is appropriate, and if so, which one.
- **`any`**: Claude must call *some* tool, but can pick which one from the tools provided.
- **`tool` with a specific `name`**: Claude is forced to call that exact tool on this turn, regardless of what the input otherwise suggests.
- **`none`**: Claude must not call a tool, even if one is provided and would normally be used.

The forcing modes (`any` / a named tool) exist for turns where the calling code already knows a tool call is the only acceptable outcome and doesn't want to leave room for Claude to just respond in prose instead — e.g. a structured-extraction step where a free-text answer would break the downstream pipeline. The common pattern is to **force a specific tool for exactly one turn, then switch `tool_choice` back to `auto`** for the rest of the conversation, once the forced step has produced the structured data it needed to produce. Leaving it forced across an entire multi-turn conversation would prevent Claude from ever responding in plain text again, which is rarely what's wanted past that first turn.

## My Insights

None of the 4 official Claude Partner Network courses document this parameter — confirmed by grepping every `claude-api/tool-use-with-claude/` chapter note (`introducing-tool-use.md`, `multi-turn-conversations-with-tools.md`, `implementing-multiple-turns.md`, `using-multiple-tools.md`, `fine-grained-tool-calling.md`, `text-editor-tool.md`, `web-search-tool.md`) for `tool_choice` — zero hits. This is a genuine coverage gap, not something buried in a chapter Bruno skimmed. It surfaced only via the `claudecertificationguide-mockexam-try1-2026-08-16` mock exam and its `/exam-analysis` writeup, which found the force-then-switch-to-auto pattern was *known* (nailed Q45 with full confidence) but *inconsistently applied* (missed the identical mechanism on Q43 just beforehand, guessed wrong on Q20 while explicitly unsure) — see `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md`, failure pattern 4. That's a recall-under-pressure problem, not a from-scratch concept gap, which is why this note exists to pin the fact down rather than to re-derive it.

Structurally this is a cousin of [retry-with-error-feedback.md](../structured-data-extraction/retry-with-error-feedback.md): both are about constraining Claude's response shape on a specific turn when free-form output isn't acceptable — `tool_choice` forces the *format* (must call this tool) up front, retry-with-error-feedback corrects the *content* after the fact when the free-form (or tool-shaped) output came back wrong. Different mechanism, same underlying goal of getting a pipeline-safe structured result out of a non-deterministic model.

## Ideas

(none yet)

## Challenges

- The exam pattern (know the fact, misapply it under time pressure on adjacent questions) suggests this isn't well suited to pure fact-recall flashcards — it may need scenario-style drilling ("here's a turn where the pipeline needs guaranteed tool-call output — what `tool_choice` value and for how long?") rather than a definition card.

## Actions

- [ ] Review this gap-topic note and add personal insights (owner: bruno)

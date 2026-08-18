## Summary

The Claude API's `tool_choice` parameter controls whether/how Claude is required to call a tool on a given request, separately from which tools are made available. It takes a few forms:

- **`auto`** (the default): Claude decides for itself whether calling a tool is appropriate, and if so, which one.
- **`any`**: Claude must call *some* tool, but can pick which one from the tools provided.
- **`tool` with a specific `name`**: Claude is forced to call that exact tool on this turn, regardless of what the input otherwise suggests.
- **`none`**: Claude must not call a tool, even if one is provided and would normally be used.

The forcing modes (`any` / a named tool) exist for turns where the calling code already knows a tool call is the only acceptable outcome and doesn't want to leave room for Claude to just respond in prose instead — e.g. a structured-extraction step where a free-text answer would break the downstream pipeline. The common pattern is to **force a specific tool for exactly one turn, then switch `tool_choice` back to `auto`** for the rest of the conversation, once the forced step has produced the structured data it needed to produce. Leaving it forced across an entire multi-turn conversation would prevent Claude from ever responding in plain text again, which is rarely what's wanted past that first turn.

### `auto` vs. `any`, concretely

Both let Claude use tools — the difference is whether responding in plain text is still a legal outcome for the turn.

- **`auto`**: Claude looks at the tools it's given and decides, on its own judgment, whether *any* tool call is warranted at all. If the input doesn't call for one, Claude just replies in prose — no tool call happens, even though tools were available.
- **`any`**: Claude *must* call one of the provided tools this turn — a plain-text-only response is off the table. It still picks *which* tool based on judgment, but "just answer in words" isn't a legal outcome.

Same three tools (`search_db`, `send_email`, `escalate`) available, two different inputs:
- User says "thanks, that's all I needed" → under `auto`, Claude just says "you're welcome" (correctly judges no tool call is needed). Under `any`, Claude is forced to call *something* anyway — it'll pick whichever tool seems least wrong (e.g. `escalate` with a nonsense payload), because "no tool call" isn't allowed. That's usually a bad outcome, and it's exactly why `any` is reserved for turns where the pipeline author already knows a tool call is the only acceptable result.
- User says "look up order #4021" → `auto` and `any` behave identically, since Claude would choose `search_db` regardless.

Rule of thumb: `auto` = "decide *if* a tool call is warranted, and if so which one." `any` = "a tool call is happening no matter what, only decide which one." Named-tool forcing goes one step further than `any` by pinning down *which* tool too, removing even that last choice.

## Scenarios (self-check)

The exam pattern here isn't "doesn't know the fact" — it's "knows the fact, picks the wrong option when it's dressed up as an adjacent mechanism." These are written the way the real distractors bite: each pairs the right answer against something that sounds architecturally reasonable but is either the wrong `tool_choice` value or the wrong scope (one-turn vs. whole-conversation).

1. **A support agent takes a free-form customer message and must always end the turn by calling `log_ticket` — no exceptions, and it's the only tool available this turn.** Would you reach for `any` or a named tool? → Either works here since there's only one tool, but a named tool (`{"type": "tool", "name": "log_ticket"}`) is the more precise choice when you want to guarantee *that specific* tool rather than "some tool, whichever Claude judges best." `any` is for when multiple tools are offered and any of them would be an acceptable outcome, not for pinning down one exact tool.

2. **A pipeline offers three tools (`search_db`, `send_email`, `escalate`) and the turn's requirement is just "don't let Claude respond in plain prose, but let it judge which of the three is right."** → `any`, not a named tool. A named tool over-constrains here (forces one specific tool regardless of which is actually appropriate); `any` gets you the "must call a tool" guarantee while preserving Claude's judgment on *which* one — that judgment is exactly what the scenario needs.

3. **A structured-extraction step forces `tool_choice: {"type": "tool", "name": "emit_invoice_fields"}` for turn 1 to guarantee parseable output. The pipeline needs three more conversational turns afterward to clarify ambiguous fields with the user in plain language.** What should `tool_choice` be set to on turns 2-4? → `auto`. This is the exact trap: leaving the forced value in place past the turn that needed it. Forcing is scoped to the one turn with the "must be structured" requirement; once that's satisfied, revert to `auto` or Claude can never again respond in prose, which breaks the clarification turns that follow.

4. **A request has `tool_choice: "none"` set, and one tool is defined and would clearly be the right call for the user's question.** What happens? → Claude does not call the tool, even though it's available and would normally be used — `none` overrides "would normally be used." This is the value most likely to be conflated with just not passing any tools at all; the distinction is that `none` still declares the tool present in the schema (useful if you want Claude reasoning about it without invoking it, e.g. explaining what it *would* do), whereas omitting tools entirely removes that context.

5. **Contrast test:** a turn produces a tool call, but the tool's *result* comes back malformed (e.g. an amount field that doesn't parse as a number) — is the fix here a `tool_choice` change, or something else? → Something else: this is retry-with-error-feedback territory ([retry-with-error-feedback.md](../structured-data-extraction/retry-with-error-feedback.md)), not `tool_choice`. `tool_choice` governs *whether/which tool gets called* on a turn, before the call happens. It has no mechanism for correcting a bad *result* after the call already happened — that's a separate retry turn with the specific validation error attached. Conflating "force the right call shape" with "fix a bad call result" is the kind of adjacent-mechanism mix-up the mock exam caught.

## My Insights

None of the 4 official Claude Partner Network courses document this parameter — confirmed by grepping every `claude-api/tool-use-with-claude/` chapter note (`introducing-tool-use.md`, `multi-turn-conversations-with-tools.md`, `implementing-multiple-turns.md`, `using-multiple-tools.md`, `fine-grained-tool-calling.md`, `text-editor-tool.md`, `web-search-tool.md`) for `tool_choice` — zero hits. This is a genuine coverage gap, not something buried in a chapter Bruno skimmed. It surfaced only via the `claudecertificationguide-mockexam-try1-2026-08-16` mock exam and its `/exam-analysis` writeup, which found the force-then-switch-to-auto pattern was *known* (nailed Q45 with full confidence) but *inconsistently applied* (missed the identical mechanism on Q43 just beforehand, guessed wrong on Q20 while explicitly unsure) — see `exam-prep/attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md`, failure pattern 4. That's a recall-under-pressure problem, not a from-scratch concept gap, which is why this note exists to pin the fact down rather than to re-derive it.

Structurally this is a cousin of [retry-with-error-feedback.md](../structured-data-extraction/retry-with-error-feedback.md): both are about constraining Claude's response shape on a specific turn when free-form output isn't acceptable — `tool_choice` forces the *format* (must call this tool) up front, retry-with-error-feedback corrects the *content* after the fact when the free-form (or tool-shaped) output came back wrong. Different mechanism, same underlying goal of getting a pipeline-safe structured result out of a non-deterministic model.

## Ideas

(none yet)

## Challenges

- The exam pattern (know the fact, misapply it under time pressure on adjacent questions) suggested this wasn't well suited to pure fact-recall flashcards — addressed above with the `## Scenarios` section, which drills the `any` vs. named-tool vs. `none` distinctions and the one-turn-vs-whole-conversation scoping mistake directly, rather than a definition card.

## Actions

- [x] Review this gap-topic note and add personal insights (owner: bruno)
- [x] Add scenario-style drills to close the recall-under-pressure gap flagged above (owner: claude)

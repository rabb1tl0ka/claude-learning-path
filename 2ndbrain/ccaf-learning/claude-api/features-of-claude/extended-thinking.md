## Source
- [Extended thinking](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287773)

## Summary

- Extended thinking gives Claude time to reason about a query before generating a final response — many chat UIs surface this as a separate, optionally-visible thinking process. It improves accuracy on complex tasks but costs more (charged for thinking-phase tokens) and adds latency.
- Deciding when to enable it: rely on prompt evals. Write a prompt, run an eval, and only consider enabling thinking if accuracy falls short after you've already optimized the prompt itself.
- Response shape changes: a **thinking block** appears alongside the usual text block, containing the reasoning text and a cryptographic **signature** — this ensures the thinking text isn't tampered with before being sent back to Claude in a future turn (unmodified thinking content is relied on heavily during generation; letting developers edit it could steer Claude unsafely).
- Occasionally Claude returns a **redacted thinking block** instead — encrypted thinking content, returned when an internal safety system flags the reasoning. It preserves context for future turns without exposing the flagged text.
- Implementation: add `thinking` (default `false`) and `thinking_budget` (minimum 1,024 tokens) to the chat function; `max_tokens` must exceed `thinking_budget` (leaving room for the actual text output), so `max_tokens` is usually set well above the thinking budget.
- For testing redacted-thinking handling, a special literal string (`"entropic, magic string, triggered, redacted thinking..."`) reliably forces Claude to return a redacted thinking block.

## My Insights

- Reacted with surprise ("Jesus.") on seeing the thinking block + signature appear in a real response for the first time.
- Questioned whether extended thinking is available in Claude Code, not just the raw API.
- Restated the eval-then-enable-thinking workflow and flagged it as feeling inefficient (an extra round trip per prompt), asking for confirmation or correction.

## Ideas

- None this session.

## Challenges

- Whether Claude Code exposes extended thinking, and how.
- Whether the "run without thinking → eval → redo with thinking if needed" workflow really is as inefficient as it sounds, or whether he's missing something about how it's actually used in production.

## Actions

- [x] Confirm whether extended thinking is available in Claude Code (owner: claude) (done same session, see `research/extended-thinking-in-claude-code.md`)
- [x] Confirm/correct the eval-then-enable-thinking workflow understanding (owner: claude) (done same session, see `research/eval-then-enable-thinking-workflow.md`)

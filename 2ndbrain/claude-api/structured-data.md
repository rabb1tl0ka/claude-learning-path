## Source
- [Structured data](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287732)
- [Structured data exercise](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287729)
- [Quiz on accessing Claude with the API](https://anthropic.skilljar.com/claude-with-the-anthropic-api/289117)

## Summary
When asking Claude for structured output (JSON, code, bulleted lists), it tends to add unwanted headers/footers/commentary — bad when the UI expects to display or copy the raw content directly (e.g. a "copy JSON" button in an app that generates AWS EventBridge rules).

**Fix: combine two techniques**
1. **Prefill the assistant message** with the start of the expected output (e.g. ` ```json `) — this "injects" the beginning of Claude's own reply, guiding it to continue from there rather than starting fresh with commentary.
2. **Stop sequence** matching the closing delimiter (e.g. ` ``` `) — halts generation right when the structured content is complete.

Leftover formatting (newlines, etc.) can be cleaned up afterward with `.strip()` or `json.loads()`.

**Exercise**: generate three sample AWS CLI commands, using *only* message prefilling + stop sequences (no prompt changes) to get all three in a single clean block with no commentary.

**Exercise walkthrough / gotchas:**
- Prefilling with three backticks alone isn't enough — Claude still inserts a language identifier (e.g. `bash`) after the backticks, since it's writing a markdown code block. Fix: include the language in the prefill yourself (` ```bash `).
- Even then, Claude may still emit only one command (implying separate code blocks) or add inline bash comments — both need to be handled.
- Most reliable fix: make the prefill more directive, not just delimiter characters — e.g. prefill with `here are all three commands in a single block without any comments:\n\`\`\`bash\n`. Prefilling text this way effectively guides Claude's continuation, not just its formatting.

Session ended with the "Accessing Claude API" quiz — 8/8, covering minimum request requirements, tokenization, streaming for slow UX, secure API key storage (server-side only), system prompts for consistent-answer use cases, low temperature for factual accuracy, and prefill+stop-sequences for raw JSON.

## My Insights
The core insight on why this works: prefilling doesn't just add characters, it injects what Claude's own reply would look like so far — Claude then treats it as its own output and continues from there, rather than treating it as an instruction to interpret.

Hit a real blocker trying to run this exercise myself: no working Anthropic SDK access yet (Bedrock account access still pending, AWS-288). Ended up asking Claude Code directly whether the Claude Code SDK could substitute for the raw Messages API here — turns out it can't, since the Claude Code SDK doesn't expose `stop_sequences` (it's built for agentic tool-use, not raw prefill/stop-sequence control). See also: `aws-bedrock-setup.md`.

## Ideas

## Challenges
Couldn't actually execute the exercise this session due to missing Bedrock access — will need to come back and run it once that's sorted (tracked in `aws-bedrock-setup.md`).

## Actions

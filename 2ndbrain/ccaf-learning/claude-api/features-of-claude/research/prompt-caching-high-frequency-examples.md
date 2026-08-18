Research note, requested by Bruno during the [prompt-caching](../prompt-caching.md) session: what real examples actually fit the "same content sent extremely frequently" case where prompt caching pays off?

## Examples

- **Large fixed system prompt + tool schemas in an agentic app** — an assistant that sends the same big system prompt and tool definitions on every single API call in its loop. These rarely change between calls, so caching them means only the new conversation turns get processed fresh each time.
- **Multi-turn chat conversations** — the growing conversation history is identical from one turn to the next except for the newest message. Caching the prefix (everything up to the last turn) means each new turn only pays for processing the new message, not the whole history again.
- **RAG/document Q&A over one fixed document or context** — a chatbot answering many different questions about the same uploaded document or knowledge-base excerpt within a session. The document text stays constant across requests while only the question changes.
- **Coding assistants working in one codebase/session** (like Claude Code itself) — the same repo context/file contents get resent across many successive turns in a session; caching avoids reprocessing that context on every turn.
- **Customer support bots with a shared policy/knowledge document** — many different incoming tickets all reference the same long policy text or FAQ content as context.

The common thread: a large, unchanging block of content (system prompt, tool schemas, a reference document, or growing-but-shared conversation prefix) that's reused across many requests within the same 1-hour cache window — as opposed to one-off requests with unique content each time, where caching has nothing to reuse.

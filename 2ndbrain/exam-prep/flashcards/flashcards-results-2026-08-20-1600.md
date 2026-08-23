# Flashcards results — 2026-08-20

**Scope:** Tool Use with Claude (all 8 chapters)
**Score:** 10/10

## Q1 — Tool-choice/tool-granularity confusion

**Question:** Two tools overlap in what they can do (e.g. one tool sets a reminder for 'a date' and another sets one for 'a duration from now'), and Claude picks the wrong one under time pressure. What's actually wrong here?

**Options:**
- The descriptions need to be longer and more detailed
- Overlapping responsibility/granularity — the tools should be redesigned so each owns a distinct, non-overlapping job (✅ correct, chosen)
- A verification pass should check which tool was called and retry if wrong
- Claude needs a few-shot example added showing the correct tool for each case

**Result:** Correct.

**Explanation:** This is about tool granularity, not wording. When two tools overlap in responsibility, no amount of description-tuning fixes it reliably — the fix is redesigning tool boundaries so each tool owns a distinct job (e.g. merge them, or make the split unambiguous like "absolute date" vs "relative duration" as separate required-parameter shapes rather than separate overlapping tools).

## Q2 — Tool description quality

**Question:** Claude is repeatedly calling the wrong tool out of two similarly-named tools in a set. Per the course's guidance on tool descriptions, what's the most direct fix?

**Options:**
- Add more few-shot examples of correct tool calls to the system prompt
- Lower the temperature so tool selection is more deterministic
- Rewrite both descriptions to be more distinct about what each tool does, when to use it, and what it returns (✅ correct, chosen)
- Add a second Claude call that double-checks which tool was picked

**Result:** Correct.

**Explanation:** When the tools themselves are legitimately distinct but Claude confuses them due to wording, the fix is description quality — say clearly what each does, when to reach for it, and what it returns. Contrast with Q1: that one was about tools that *are* actually overlapping in responsibility — no description rewrite fixes that, only redesign. This one assumes distinct tools with unclear descriptions.

## Q3 — Multi-block assistant messages

**Question:** When Claude decides to use a tool, what does the assistant message actually contain?

**Options:**
- Only a single tool_use block, since Claude cannot emit text on a tool-call turn
- A tool_result block confirming the tool already ran
- Potentially multiple content blocks — e.g. a text block plus a tool_use block with the tool's id, name, and input (✅ correct, chosen)
- A single plain-text string describing which tool to call

**Result:** Correct.

**Explanation:** Claude's assistant messages are structured as a list of content blocks, not a single value — a tool-call turn commonly includes a text block (Claude's reasoning/narration) alongside the tool_use block (which itself carries the tool's `id`, `name`, and `input`). That full structure has to be preserved when appending to message history, not just the extracted text.

## Q4 — Chained tool calls

**Question:** A user asks "what day is 103 days from today?" and the app has separate get_current_date_time and add_duration_to_date_time tools. Why does this require multiple tool calls in sequence rather than one?

**Options:**
- Claude always calls every available tool once regardless of relevance
- Claude first needs today's date from one tool before it has enough information to call the second tool that adds the duration (✅ correct, chosen)
- Each tool can only return one field, so results must be split across calls
- The API enforces a hard limit of one tool call per tool per conversation

**Result:** Correct.

**Explanation:** This is a genuine information dependency: Claude can't call `add_duration_to_date_time` meaningfully until it knows the starting date, so it calls `get_current_date_time` first, gets the result back, then makes the second call. This differs from "ordering between tool calls" (multiple tool_use blocks in one response) — that's about independent calls, this is about sequential dependent calls across turns.

## Q5 — run_conversation loop structure

**Question:** Inside the run_conversation function's while loop, what determines whether the loop breaks and returns a final answer versus continuing?

**Options:**
- Whether the user sends a new message
- Whether the response's stop_reason is tool_use (continue) or something else (break and return) (✅ correct, chosen)
- A fixed number of iterations set before the loop starts
- Whether the message list has grown past a fixed length

**Result:** Correct.

**Explanation:** The loop check is exactly the `stop_reason` field, applied concretely: `tool_use` → run the tool, append `tool_result`, loop again; anything else (`end_turn`, etc.) → that's the final answer, break and return it.

## Q6 — Sending all schemas every request

**Question:** The course's approach sends every tool's full schema to Claude on every request. What's the main drawback of this pattern as opposed to a lookup-tool approach?

**Options:**
- The API rejects requests with more than one tool schema
- It requires a separate API key per tool
- It's not the most efficient design — an alternative is exposing just tool names plus a lookup tool that fetches full schemas on demand (✅ correct, chosen)
- Claude cannot parse more than one schema per request

**Result:** Correct.

**Explanation:** This is a recurring insight from the notes. Sending every schema every request works but doesn't scale token-wise as the tool count grows. The alternative pattern: expose just tool names up front, plus a single lookup tool Claude calls to fetch a specific tool's full schema only once it's decided it wants to use it.

## Q7 — Why tool streaming feels laggy

**Question:** By default, why does streaming a tool call's arguments feel laggy (long pauses, then a sudden burst of text) rather than smooth?

**Options:**
- Streaming is disabled by default for any request containing tools
- The API buffers generated JSON and validates it against the tool's schema one top-level key at a time before releasing it (✅ correct, chosen)
- Claude generates tool arguments slower than regular text
- The network connection throttles tool_use events specifically

**Result:** Correct.

**Explanation:** The API validates each top-level key's value against the tool schema before releasing any of the chunks that made it up — so you get a wait (buffered + validated), then a burst (all buffered chunks arrive nearly simultaneously), then the cycle repeats for the next key. `fine_grained=True` turns this validation off in exchange for smoother streaming, at the cost of your code needing to tolerate malformed JSON.

## Q8 — Two halves of every tool

**Question:** For the text editor tool, which half does Anthropic provide, and which half is still the developer's responsibility?

**Options:**
- Anthropic provides neither half; both must be written from scratch
- Anthropic provides the JSON schema; the developer must still implement the function that performs the real file operations (✅ correct, chosen)
- Anthropic provides both halves, requiring no developer work at all
- Anthropic provides the full implementation; the developer only needs to register the schema

**Result:** Correct.

**Explanation:** Every tool has two halves: schema (tells Claude the tool exists and its arguments) and implementation (the code that actually runs). For the text editor tool, Anthropic ships only the schema half by default — the implementation touches your filesystem, so it has to be yours (the course's demo notebook provides a working reference class for this).

## Q9 — Why some tools are server-side only

**Question:** Why can Anthropic safely run web search entirely on its own infrastructure, while file edits and bash commands must stay client-side?

**Options:**
- Web search is a lighter-weight operation that requires less compute
- Anthropic does not have infrastructure capable of running file operations
- Web search never needs to touch anything private to the user, while file edits and bash commands are meaningless without access to the user's own environment (✅ correct, chosen)
- Client-side tools are always deprecated in favor of server-side ones

**Result:** Correct.

**Explanation:** This is the server-tool vs. client-tool divide: web search is inherently public data, so Anthropic can run the whole thing on their infra with zero access to the user's environment. File edits and bash commands are meaningless without touching your specific filesystem/environment, so no matter how "built-in" the schema looks, the execution has to stay on your side.

## Q10 — Batch tool vs Message Batches API

**Question:** A quiz question asks what problem the "batch tool" solves. What's the important nuance here?

**Options:**
- The batch tool automatically retries failed tool calls
- There's no literal "batch tool" in the tool-use material — the question actually refers to the separate Message Batches API for bulk async processing of independent requests (✅ correct, chosen)
- The batch tool is only available in streaming mode
- The batch tool is a built-in tool exactly like the text editor tool, with its own JSON schema

**Result:** Correct.

**Explanation:** Same nuance flagged in the quiz-review chapter note: the material never actually teaches a "batch tool"; the question is really pointing at the Message Batches API (bulk async processing of many independent requests), a distinct mechanism from reducing tool round-trips within one conversation.

## Weakest topics this session

None missed this session (10/10). Existing weak spots outside this session's scope remain in `.flashcards/weak-spots.md` — notably `few-shot-vs-tool-granularity` (Prompt Engineering Techniques, 100% wrong) and `worktree-coordination` (Long-running sessions, 100% wrong).

# Agentic Loops

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-1-agentic-loops

## Summary

### Core Definition
An agentic loop represents "the core execution cycle behind every Claude-based agent" implemented as deterministic control flow in code. It is explicitly not a prompt trick, retry loop, or chatbot turn mechanic.

### The Four-Step Lifecycle
The loop repeats these steps until completion:

1. **Send Request**: Transmit a message to Claude via the Messages API, including system prompt, conversation history, and prior tool results.
2. **Inspect `stop_reason`**: Read the authoritative signal field from the response. Two primary values:
   - `"tool_use"` — Claude requests tool execution; loop continues
   - `"end_turn"` — Claude has completed work; loop terminates
3. **Handle `tool_use`**: Execute the requested tool(s), append tool results to conversation history as a new message, and resend the updated conversation to Claude.
4. **Handle `end_turn`**: Present the final response to the user.

**Critical Detail**: Step 3 frequently fails in practice. Tool results "must be appended to conversation history." Without this, Claude cannot reason about new information on subsequent iterations since it never receives what the tool returned.

### Authoritative Loop Control
"The `stop_reason` field is the only reliable signal for loop control. It is deterministic and unambiguous." Never use natural language parsing, text content checks, or arbitrary iteration caps as the primary stopping mechanism.

#### Additional stop_reason values in production
While the exam focuses on `tool_use` and `end_turn`, the live Messages API returns additional values requiring handling:
- `pause_turn` — continue long-running server-tool operations
- `max_tokens` — response filled token limit
- `stop_sequence` — sequence boundary reached
- `refusal` — model declined (current models like Fable 5)
- `model_context_window_exceeded` — context window filled; handle as truncation

Guidance: treat any value other than `end_turn` as "not finished, check why" rather than assuming it must be `tool_use`.

### Model-Driven vs. Pre-Configured Decision-Making
- **Model-driven decision-making**: Claude reads the task, weighs available tools, and selects one. Flexes to unmapped situations and edge cases.
- **Pre-configured decision trees/fixed sequences**: Developer hard-codes which tool runs when. Less adaptive.

The exam favors model-driven approaches except when "business logic demands deterministic compliance — financial operations, security checks, regulatory requirements — programmatic enforcement overrides that flexibility."

### Three Anti-Patterns for Loop Termination

**Anti-Pattern 1: Parsing Natural Language Signals**
Checking if Claude said "I'm done" or "task complete" is wrong because natural language is inherently ambiguous. Claude might say "I've finished analysing the first file" while intending to continue. `stop_reason` exists to eliminate this ambiguity.

**Anti-Pattern 2: Arbitrary Iteration Caps as Primary Stopping Mechanism**
Setting "stop after 10 loops" as the main termination method is wrong: it either cuts off useful work (task genuinely needs 12 iterations) or runs unnecessary iterations (task finishes in 3). The model signals completion via `stop_reason`. Iteration caps are acceptable as a safety net (max bound to prevent runaway agents), never as primary control.

**Anti-Pattern 3: Checking for Assistant Text Content as Completion Indicator**
Using `response.content[0].type == "text"` to decide the loop is finished is wrong because Claude can return text alongside `tool_use` blocks. A response might contain explanatory text ("I'll now search for the customer's order history") immediately followed by a tool call. Text presence does not indicate completion.

### Practical Example: The Premature Termination Bug
A customer support agent works for simple queries but stops mid-task on complex requests. The code checks `if response.content[0].type == "text"` to determine completion.

The bug: Claude returns text ("Let me look up your order") alongside a `tool_use` block requesting the `lookup_order` tool. The code sees text at position [0], concludes the agent is finished, and returns an incomplete response.

The fix: replace the content-type check with `stop_reason` verification. Continue the loop when `stop_reason == "tool_use"`, terminate when `stop_reason == "end_turn"`. This works regardless of content types in the response.

### Exam Traps (Explicitly Called Out)
1. Using `response.content[0].type == 'text'` — Claude can return text alongside tool_use blocks; text presence does not indicate completion.
2. Setting arbitrary iteration caps (e.g., "stop after 10 loops") as primary stopping — use as safety net only.
3. Parsing natural language phrases like "I'm done" or "task complete" — ambiguous; `stop_reason` provides the deterministic signal.
4. Forcing `tool_choice` to `'any'` — forces tool use even when genuinely finished, creating infinite loops. Let the model signal completion naturally via `stop_reason`.

### Practice Scenario and Correct Answer
**Scenario**: Developer's agent terminates prematurely when Claude returns text alongside a tool call. The loop checks `response.content[0].type == 'text'` to determine finish status. Users report incomplete responses on complex queries.

**Correct Answer (Option B)**: "Check the `stop_reason` field instead of content type — continue when `stop_reason` is tool_use, terminate when end_turn"

Rejected options:
- Option A (iteration cap of 15): Does not address root cause
- Option C (tool_choice 'any'): Creates infinite loops
- Option D (parse assistant text): Reintroduces natural language ambiguity

### Build Exercise Learning Objectives
A 45-minute exercise requires building a multi-tool agent loop with:
1. Calculator tool and web search stub (mock results)
2. Agentic loop using `client.messages.create()` with `stop_reason` inspection
3. Tool execution with `tool_use` handling: extract tool calls, execute, append results to conversation history
4. `end_turn` handling: extract final text response and return
5. Multi-turn sequential testing: search first, calculate with search result, return combined answer
6. Safety iteration cap of 20 (maximum bound, not primary mechanism) with warning logging

The exercise emphasizes that normal queries should terminate via `stop_reason` well before reaching 20 iterations.

### Key Takeaway for Exam Preparation
Memorize that `stop_reason` is the authoritative, deterministic control signal. Reject all alternatives (content-type checks, natural language parsing, iteration caps as primary mechanism) as anti-patterns. The agentic loop is not a prompt trick — it is code-based, deterministic control flow.

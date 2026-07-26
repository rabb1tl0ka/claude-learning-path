# System Prompt: Claude API vs Claude Code SDK

Comparison done while taking the "Building with the Claude API" course (System Prompts video, Accessing Claude with the API chapter), triggered by wondering whether the same system prompt customization applies to the Claude Code SDK (used in `claude-aitpm`).

## Side-by-side

| | **Claude API (Messages API)** | **Claude Agent SDK / Claude Code SDK** |
|---|---|---|
| **Parameter** | `system` (top-level field in the request body) | TS: `systemPrompt` in `options` · Python: `system_prompt` in `ClaudeAgentOptions` |
| **Default if unset** | None, empty system prompt unless you provide one | Claude Code's built-in harness prompt (tool conventions, coding guidance, etc.) |
| **Modes available** | One mode: pass a string, that's your entire system prompt | Three modes: (1) plain string = full override, (2) `{type:"preset", preset:"claude_code"}` = keep default, (3) preset + `append` = default + your addition |
| **What gets replaced** | Whatever you put in `system` is literally the full system prompt sent to the model | Mode 1 replaces Claude Code's harness prompt entirely; mode 3 layers on top of it instead |
| **Effect on tool-use conventions** | N/A, you control tools/prompt yourself from scratch, there's no built-in harness | Mode 1: you lose Claude Code's built-in tool-use style/conventions (tools still callable, just no scaffolding around them) unless you write your own. Mode 3: conventions preserved |
| **CLAUDE.md / project instructions** | Not applicable, this concept doesn't exist at the raw API level | Loads and injects regardless of `systemPrompt` mode, it's delivered separately into the conversation, not via the system prompt |
| **Model-level guardrails (RLHF/Constitutional AI)** | Unaffected, baked into the model, not touchable via `system` | Unaffected, same as API, no SDK mode changes this |
| **Typical use case** | Building a fully custom agent/product from a blank slate | Layering your own instructions on Claude Code's existing coding-agent behavior (e.g. `append` mode for something like `claude-aitpm`) |

## Key distinction: harness prompt vs model-level guardrails

Two separate layers are at play, and "Full Override" in the Claude Code SDK only touches one of them:

1. **Model-level guardrails**, baked in during training (RLHF / Constitutional AI). Not text sitting in an invisible system prompt, it's part of the model's trained behavior. No API or SDK parameter strips this, override mode or not.
2. **Harness-level system prompt**, this is what "Full Override" touches. Claude Code's default system prompt is just text instructions (tool-use conventions, coding style guidance, "you are Claude Code..." framing) sent as the `system` field of the underlying Messages API call. Override mode replaces that text with your own, same mechanism as the raw API's `system` param.

So "Full Override" swaps the wrapper's instructions, not the model's trained-in behavior. Even in override mode you're still calling the same Sonnet/Opus/Haiku model with its RLHF-trained guardrails and default persona tendencies intact, because those aren't system-prompt-resident, they're model-resident.

Nuance: a system prompt can still shift behavior at the margins (e.g. "respond only in JSON" measurably changes output style, and crafted prompts can partially erode safety behavior in edge cases per adversarial research), but that's far from "no guardrails." Trained safety behavior isn't something a system prompt param turns off.

## Sources
- [Modifying System Prompts](https://code.claude.com/docs/en/agent-sdk/modifying-system-prompts.md)
- [TypeScript SDK Reference](https://code.claude.com/docs/en/agent-sdk/typescript.md)

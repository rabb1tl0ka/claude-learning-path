Research note, requested by Bruno while discussing `input_schema` in tool definitions: why does a JSON Schema need an extra `"strict": true` flag to be enforced, when defining a schema at all implies you want it respected?

## Findings

`input_schema` on a tool definition is a standard JSON Schema object (`type`, `properties`, `required`, `enum`, nested objects/arrays — no Anthropic-specific dialect). But by default, that schema is only used as **guidance fed into the model's generation**, not as a hard constraint enforced by the API.

**Without `strict: true`:** the schema gets folded into the prompt as a strong hint. Claude generates the `tool_use.input` JSON the same way it generates any other text — token by token, best-effort, aiming to match the described shape. It's usually very good at this, but on complex/nested schemas it can still drop a field, pick a wrong enum value, or produce a type mismatch. This is generation, not validation.

**With `strict: true`:** the API switches to **constrained decoding** — at each token, only tokens that keep the output on a valid path through the schema are allowed. It's not "generate then check," it's "structurally impossible to leave the schema." That's a different mechanism under the hood, which is why it's an opt-in flag rather than the default behavior of merely having a schema.

Why it isn't the default:
- Constrained decoding carries a latency/compute cost per request.
- It comes with schema restrictions — requires `additionalProperties: false` and `required` to be set — not every schema is eligible as-is.
- Best-effort schema-following is good enough for most tool-use cases, so paying the strict-mode cost by default for everyone doesn't make sense.

**Where `strict` lives:** it's a top-level field on the tool definition itself, alongside `name`, `description`, and `input_schema` — not a field on `tool_choice`.

## Mental model

- Schema without `strict`: a strong hint to the model.
- Schema with `strict`: a hard guarantee enforced by the decoding process itself.

Same JSON Schema object either way — the difference is entirely in how much the API enforces it.

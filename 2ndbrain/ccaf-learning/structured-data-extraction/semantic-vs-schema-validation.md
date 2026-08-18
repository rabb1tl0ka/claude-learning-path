## Summary

Structured extraction pipelines need two distinct layers of validation, and conflating them is a common design mistake.

**Schema (type) validation** confirms that each extracted field has the *shape* it's supposed to have: is `invoice_date` a valid date string, is `amount` a number, is `line_items` an array of objects with the right keys. This is what Claude's tool use JSON Schema enforcement handles at the API boundary, and what a library like Pydantic's `model_validate()` re-checks on the application side once the data lands in code (or, in the newer Claude API "structured outputs" feature, guarantees are enforced directly against a JSON Schema — including one generated from a Pydantic model — so the response is schema-valid coming out of the API). Schema validation catches things like: a string where a number was expected, a missing required key, an out-of-range enum value. It is purely structural — it has no concept of whether the *content* makes sense.

**Semantic (business-rule) validation** is a separate layer entirely: does the content that passed schema validation actually make logical/business sense. Classic example: an invoice's line items are individually well-typed numbers, and the stated total is also a well-typed number, but the line items sum to £450 while the stated total says £500. Schema validation sees two perfectly valid numbers and passes the extraction. Only an explicit cross-field business-rule check — "do the line items sum to the stated total" — catches the discrepancy. Other examples: a shipment date that comes before an order date, a percentage field outside 0-100, a required field that's technically present but obviously a placeholder/fabricated value.

The practical implication: **schema validation passing is not evidence that an extraction is correct** — it only proves the extraction is well-formed. Teams that stop at schema validation (e.g. "it validated against the Pydantic model, ship it") will still let semantically broken extractions through to production. Catching those requires explicit, hand-written cross-field validators (in Pydantic, this is typically a `@model_validator` that runs after individual field validation and checks relationships between fields) — and those validator error messages are exactly what feeds the retry-with-error-feedback loop (see `retry-with-error-feedback.md`), since a generic "validation failed" isn't actionable but "line items sum to £450, stated total is £500" is.

## My Insights

Directly connects to Claude's tool use JSON Schema enforcement covered in `claude-api/tool-use-with-claude/introducing-tool-use.md` and fine-grained tool calling — that's exactly the schema-validation half of this picture (Claude's structured outputs / strict tool use guarantees the *shape*). The semantic-validation half (does the content make business sense) is the genuinely new idea the official courses don't cover — it has to be hand-built as separate application logic regardless of how strict the schema guarantee is.

## Challenges

- Business rules are domain-specific and have to be enumerated by hand for each extraction schema — there's no generic way to auto-derive "line items must sum to total" from a schema definition, so this validation layer scales with effort, not with tooling.
- Some semantic errors are genuinely ambiguous (e.g. rounding differences of a few cents in a total) and need a tolerance/threshold built into the validator rather than an exact-match check, which itself requires domain judgment to set correctly.

## Actions

- [ ] Review this gap-topic note and add personal insights (owner: bruno)

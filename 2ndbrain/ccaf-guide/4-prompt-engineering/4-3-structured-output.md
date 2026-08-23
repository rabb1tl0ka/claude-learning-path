# Structured Output with Tool Use

## Source
https://claudecertificationguide.com/learn/4-prompt-engineering/4-3-structured-output

## Summary

### Core Reliability Hierarchy

The lesson establishes a definitive ranking for obtaining schema-compliant structured output from Claude:

1. **tool_use with JSON schemas** — eliminates JSON syntax errors entirely
2. **Prompt-based JSON** — model can produce malformed JSON

This hierarchy is characterized as essential exam material: "Commit this hierarchy to memory. The exam builds on it."

The mechanism works through constraint: JSON schemas embedded in tool definitions restrict Claude's response structure, preventing syntax issues like missing brackets, trailing commas, or unquoted keys. Conversely, requesting JSON in text responses provides "no structural guarantees and will periodically produce unparseable output in production."

### The tool_choice Parameter: Three Critical Modes

The `tool_choice` parameter governs whether and how Claude invokes tools. The three modes represent a core exam distinction:

**Mode 1: "auto" (default)**
Claude decides autonomously whether to call a tool or return text. The model may respond conversationally instead of invoking the extraction tool. Appropriate when "the model legitimately needs the option to respond conversationally."

**Mode 2: "any"**
Claude MUST call a tool but selects which one independently. Designed for scenarios with multiple extraction schemas (extract_invoice, extract_receipt, extract_contract) where document type is unknown. Guarantees structured output while preserving tool flexibility.

**Mode 3: Forced Selection**
Configuration: `{"type": "tool", "name": "extract_metadata"}` — Claude MUST call this specific named tool. Enforces mandatory first steps (e.g., ensuring metadata extraction precedes enrichment). Provides "no flexibility, maximum control."

### Critical Limitation: What tool_use Does NOT Prevent

The lesson emphasizes exam deception potential here. tool_use with JSON schemas eliminates **syntax** errors exclusively but does NOT prevent **semantic** errors:

- **Sum discrepancies:** Line items failing to sum to stated totals
- **Field placement errors:** Values in wrong fields (e.g., dates in amount fields when both are strings)
- **Fabrication:** Model invents values for required fields when source documents lack information

The distinction is absolute: "The schema guarantees structure. It doesn't guarantee correctness." In other words, a JSON schema (via `tool_use`) constrains the *shape* of the output — which keys exist, what types they hold, whether required fields are present — but it has no mechanism to verify that the *content* placed into those fields is factually or arithmetically correct. The schema validator will happily accept a syntactically perfect JSON object whose `stated_total` field contains a fabricated number, or whose `invoice_date` field contains a plausible-looking but wrong date, because nothing about JSON schema conformance checks against the source document's actual content. Catching that class of error requires a separate semantic-validation layer (see 4-4), not tighter schemas.

### Schema Design for Production: Specific Patterns

**Optional/Nullable Fields Pattern**
When source documents may lack certain information, make those fields optional or nullable. This represents "the primary defence against fabrication." Required fields create pressure to produce values even when absent from source. Nullable fields permit honest `null` returns.

Example schema structure provided:
```json
{
  "type": "object",
  "properties": {
    "invoice_number": { "type": "string" },
    "vendor_name": { "type": "string" },
    "payment_terms": { "type": ["string", "null"] },
    "purchase_order": { "type": ["string", "null"] }
  },
  "required": ["invoice_number", "vendor_name"]
}
```

The required array contains only fields always present; optional fields use `["string", "null"]` syntax.

**Unclear Enum Value Pattern**
For genuinely ambiguous cases, include explicit "unclear" option in enum fields. This "prevents the model from forcing a classification when the evidence is ambiguous."

**Other + Detail String Pattern**
Include "other" enum value paired with freeform detail string field. Captures edge cases not covered by predefined categories.

Example provided:
```json
{
  "category": {
    "type": "string",
    "enum": ["invoice", "receipt", "contract", "unclear", "other"]
  },
  "category_detail": {
    "type": ["string", "null"],
    "description": "Freeform detail when category is 'other'"
  }
}
```

**Format Normalisation Rules**
Include format instructions in prompt alongside schema. Schema enforces structure; prompt enforces consistency (e.g., "All dates in ISO 8601 format," "All currency amounts as decimal numbers without currency symbols").

### Exam Traps Explicitly Called Out

**Trap 1: Overestimating tool_use Scope**
Belief: tool_use with JSON schemas prevents all extraction errors.
Reality: Eliminates JSON syntax errors only. Semantic errors persist.

**Trap 2: Confusing tool_choice Modes**
Conflating 'auto' with 'any':
- 'auto' allows text returns instead of tool calls — no guaranteed structured output
- 'any' guarantees tool call but model chooses which tool
- For guaranteed structured output with unknown document types, use 'any'

**Trap 3: All Required Fields Misconception**
Making all schema fields required to ensure completeness actually causes fabrication. Required fields pressure models to invent values for missing information. Optional/nullable fields are "always preferable to plausible-looking fabricated data."

### Practice Scenario and Answer

**Scenario:** Extraction system using tool_use with all required fields reports model inventing plausible dates and amounts when documents lack this information.

**Correct Answer:** "Make fields optional or nullable when source documents may not contain the information" (Option A)

Incorrect alternatives:
- Option B (switch to prompt-based JSON) — reverses the reliability hierarchy
- Option C (add hallucination instruction) — doesn't address structural pressure
- Option D (post-extraction validation) — addresses symptom, not cause

### Build Exercise Requirements

Five-step exercise validates practical understanding:

1. **Schema design** with 3 required fields, 3 optional/nullable fields, enum including unclear/other, and detail string. Success criterion: valid JSON schema with required array containing only always-present fields.

2. **Test tool_choice auto** to observe text-only responses. Success criterion: at least one response returning text description instead of tool call.

3. **Switch to tool_choice any** and verify all responses use structured output. Success criterion: all responses with `stop_reason` of `tool_use` containing valid tool calls; no text-only responses.

4. **Force specific tool** with `{"type": "tool", "name": "extract_metadata"}`. Success criterion: response always calls specified tool regardless of appropriateness.

5. **Process 5 documents** (3 complete, 2 missing fields) and verify nullable fields return `null` rather than fabricated values. Success criterion: complete documents fully populated; documents with missing information show `null` in nullable fields with no invented dates, amounts, or identifiers.

### Source Attribution

Study materials derived from Claude Certified Architect Foundations Exam Guide (Task Statement 4.3), Anthropic's Tool Use documentation, and Building with Claude API (Skilljar platform).

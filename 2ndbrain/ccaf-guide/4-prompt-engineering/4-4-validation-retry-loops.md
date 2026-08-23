# Validation, Retry, and Feedback Loops

## Source
https://claudecertificationguide.com/learn/4-prompt-engineering/4-4-validation-retry-loops

## Summary

### Core Premise

Production extraction systems fail predictably. The validation-retry pattern transforms failures into self-correcting workflows by enabling systems to learn from specific errors and re-attempt extractions with targeted guidance.

### The Retry-with-Error-Feedback Pattern

The correct retry mechanism transmits three mandatory components back to the model:

1. **Original document** — provides the source material for re-examination
2. **Failed extraction** — shows what the model previously produced
3. **Specific validation error** — communicates exactly what went wrong

The guide emphasizes this is superior to naive retries. Without the specific error message, the model lacks guidance and typically reproduces identical mistakes. With it, the model can target self-correction by re-examining the document for missed line items, checking field placement, or recalculating totals.

**Example Structure**

The guide provides this TypeScript code pattern:

```
const retryMessages = [
  {
    role: "user",
    content: `Original document:\n${originalDocument}\n\n` +
      `Your extraction:\n${JSON.stringify(failedExtraction)}\n\n` +
      `Validation error: Line items sum to £450 but stated_total is £500. ` +
      `Please re-extract, ensuring all line items are captured.`
  }
];
```

The validation error message explicitly contrasts calculated versus stated values and directs attention to the discrepancy source.

### The Retry Effectiveness Boundary

This concept receives the "most aggressive" exam testing. Retries have a clear scope:

**Retries ARE effective for:**
- Format mismatches (wrong date format, inconsistent currency notation)
- Structural output errors (values in wrong fields, incorrect nesting)
- Misplaced values (data existing in the document but extracted into wrong fields)
- Mathematical errors (missed line items affecting totals)

**Retries are NOT effective for:**
- Information genuinely absent from source documents
- Data existing only in external documents not provided to the model
- Fields requiring knowledge the model does not possess

The guide stresses: "If a document genuinely doesn't contain a department name, no amount of retrying will produce a correct value. Flag the extraction for human review, or return null if the schema allows it."

### Self-Correction Flow Design

Rather than relying solely on external validation logic, systems should build self-correction into the extraction schema itself:

**calculated_total vs stated_total Pattern**

Extract both the sum the model calculates from individual line items AND the total stated in the document. When these differ, automatic discrepancy flagging occurs without external logic.

```json
{
  "line_items": [
    { "description": "Widget A", "amount": 150.00 },
    { "description": "Widget B", "amount": 300.00 }
  ],
  "calculated_total": 450.00,
  "stated_total": 500.00,
  "total_discrepancy": true
}
```

**conflict_detected Booleans**

Add boolean fields that flag when the source document contains contradictory information. Example: if a document states "payment due: 30 days" in one section but "payment terms: net 60" in another, the model should extract both statements and set `conflict_detected: true` rather than silently selecting one interpretation.

**detected_pattern Fields**

For code review and analysis pipelines, add `detected_pattern` fields to structured findings to track which specific code construct triggered each finding.

```json
{
  "finding": "Potential SQL injection vulnerability",
  "severity": "critical",
  "detected_pattern": "string concatenation in SQL query",
  "file": "user_service.py",
  "line": 42
}
```

The guide explains: "When developers dismiss findings, you can analyse dismissal patterns by `detected_pattern`. If developers consistently dismiss findings triggered by 'variable shadowing in nested scope,' that pattern likely needs prompt refinement."

### Schema Syntax Errors vs Semantic Validation Errors

The exam deliberately distinguishes these categories:

**Schema syntax errors** — Malformed JSON, missing required fields, wrong data types. These are "eliminated entirely" by `tool_use` with JSON schemas.

**Semantic validation errors** — Correct JSON structure but incorrect values. Line items that do not sum correctly, dates with improper ordering, values in wrong fields. These require "validation logic outside the schema" and are the focus of retry loops.

The guide emphasizes: "The overlap between these task statements is intentional. The exam tests whether you understand that `tool_use` solves the first category but not the second."

### Pydantic as the Validation Layer

The exam guide explicitly names Pydantic alongside JSON Schema in the hands-on exercise: "when Pydantic or JSON schema validation fails, send a follow-up request including the document, the failed extraction, and the specific validation error."

Pydantic performs dual functions:
- **Parsing** enforces structure (types, required fields, enums)
- **Validators** enforce semantics (cross-field arithmetic, date ordering rules)

Both failure types surface through a single `ValidationError` with machine-readable, per-field error messages.

**Complete Pydantic Example**

```python
import json
from pydantic import BaseModel, ValidationError, model_validator

class LineItem(BaseModel):
    description: str
    amount: float

class Invoice(BaseModel):
    line_items: list[LineItem]
    stated_total: float

    @model_validator(mode="after")
    def totals_must_match(self):
        calculated = round(sum(i.amount for i in self.line_items), 2)
        if abs(calculated - self.stated_total) > 0.01:
            raise ValueError(
                f"line items sum to {calculated} but stated_total is {self.stated_total}"
            )
        return self

try:
    invoice = Invoice.model_validate(tool_input)
except ValidationError as e:
    errors = "\n".join(
        f"{'.'.join(map(str, err['loc'])) or 'invoice'}: {err['msg']}" for err in e.errors()
    )
    retry_message = (
        f"Original document:\n{original_document}\n\n"
        f"Your extraction:\n{json.dumps(tool_input)}\n\n"
        f"Validation errors:\n{errors}\n\n"
        f"Please re-extract, fixing the identified errors."
    )
```

The `except` block implements the retry-with-error-feedback pattern, with Pydantic supplying the third ingredient (specific error) in a format directly insertable into the prompt.

### Current SDK-Level Implementation (As of July 2026)

The Python SDK's `client.messages.parse(..., output_format=Invoice)` returns validated Pydantic instances via `parsed_output`. The `strict: true` parameter on tool definitions guarantees schema-conformant inputs server-side through Structured Outputs.

However: "Neither removes the semantic layer: the platform enforces the schema, your validators enforce the business rules, and the retry loop consumes whichever one fails."

### Exam Traps

**Trap 1: Assuming Retries Always Work for Extraction Failures**
Retries fix format mismatches, structural errors, and misplaced values. They cannot produce information genuinely absent from the source document. The exam presents both fixable and unfixable scenarios — candidates must distinguish them.

**Trap 2: Implementing Retries Without the Specific Validation Error**
"Naive retries without error feedback produce the same mistakes. The model needs to see exactly what went wrong (e.g., 'line items sum to £450 but stated total is £500') to self-correct effectively."

**Trap 3: Relying on Schema Validation Alone Without Semantic Checks**
Schema validation via `tool_use` catches syntax errors. Semantic errors (wrong sums, misplaced values, fabricated data) require validation logic and retry loops.

**Trap 4: Treating Pydantic as Redundant Once tool_use Enforces JSON Schema**
Schemas eliminate syntax errors but cannot express cross-field semantic rules like sums that must match or dates that must be ordered. Pydantic validators encode those rules and produce the specific, per-field error messages the retry loop feeds back to the model.

### Practice Scenario

The guide presents this scenario:

"Your extraction pipeline validates that line item amounts sum to the stated total. For Document A, the calculated sum is £450 but the stated total is £500. For Document B, the 'department' field is missing entirely from the source text. Which retry strategy is correct?"

**Option A:** Retry both documents with validation errors
**Option B:** Skip retries for both; flag all for human review
**Option C:** Retry Document A with the discrepancy error; flag Document B for human review since information is absent
**Option D:** Retry both documents with the same prompt, since extraction is non-deterministic

The answer is Option C, illustrating the core concept: the retry effectiveness boundary determines which failures are fixable through re-extraction versus which require human intervention.

### Build Exercise Requirements

The hands-on exercise requires implementing:

1. **Extraction tool definition** with `calculated_total` and `stated_total` number fields, `total_discrepancy` boolean, `conflict_detected` boolean, and `detected_pattern` string fields on findings

2. **Validation logic** checking field completeness, numerical consistency (calculated sum matches stated total), enum validity, and date ordering — returning specific, actionable error messages

3. **Retry loop** constructing follow-up messages containing original document, failed extraction, and specific validation error

4. **Testing with 5 documents**: 2 with fixable errors (misplaced values, wrong totals) and 3 with unfixable errors (absent information) — verifying the loop retries only fixable cases

5. **Dismissal pattern analysis** logging `detected_pattern` data and analysing which patterns are most frequently dismissed to identify prompt refinement priorities

### Key Terminology

- **Retry-with-error-feedback**: The three-component message pattern (original document + failed extraction + specific validation error)
- **Retry effectiveness boundary**: The threshold distinguishing fixable errors (format, structural, mathematical) from unfixable errors (absent information)
- **Self-correction fields**: `calculated_total`, `stated_total`, `total_discrepancy`, `conflict_detected`
- **Semantic validation**: Business rule enforcement beyond schema structure
- **Syntax validation**: Structure, type, and required field enforcement
- **model_validator**: Pydantic decorator using `mode="after"` for cross-field validation
- **Structured Outputs**: SDK feature ensuring schema-conformant inputs via `strict: true`
- **detected_pattern**: Field tracking which specific construct triggered each finding

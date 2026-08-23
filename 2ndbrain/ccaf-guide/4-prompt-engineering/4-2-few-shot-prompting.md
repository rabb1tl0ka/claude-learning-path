# Few-Shot Prompting

## Source
https://claudecertificationguide.com/learn/4-prompt-engineering/4-2-few-shot-prompting

## Summary

### Core Principle

"Few-shot examples are the most effective technique for achieving consistent, well-formatted output from Claude." This is designated as "a direct exam principle." The page emphasizes that when output proves inconsistent, few-shot examples—not additional instructions, confidence thresholds, or temperature adjustments—represent the primary intervention.

### Three Specific Deployment Triggers

The page identifies exactly when few-shot examples become necessary:

**1. Inconsistent Output Formatting Despite Detailed Instructions**
When a thoroughly specified prompt produces varying structures across invocations (sometimes bulleted lists, sometimes tables, sometimes prose), additional instructions will not resolve the problem. Few-shot examples demonstrating the exact desired format are required.

**2. Inconsistent Judgement Calls on Ambiguous Cases**
Examples provided: a code review tool flags variable shadowing as "critical" in one file and "minor" in another; a tool selection agent routes "check my order" to different tools depending on phrasing. These scenarios demand examples demonstrating correct judgement with reasoning.

**3. Extraction Tasks Producing Empty/Null Fields for Existing Information**
When data exists but appears in unexpected formats—embedded in narrative text rather than structured tables, or split across multiple paragraphs—few-shot examples showing extraction from varied document structures resolve this.

### Construction Rules: Precise Specifications

**Quantity Rule: 2-4 Targeted Examples**
"Fewer than 2 doesn't establish a pattern. More than 4 wastes tokens without proportional benefit." Examples must target specific ambiguous scenarios causing problems.

**Reasoning Requirement: Essential Component**
The page states: "Each example must show reasoning. Don't just show input-output pairs." The distinction is critical:

*Without reasoning:* The model learns only surface-level associations ("queries mentioning order numbers go to lookup_order").

*With reasoning:* The model learns generalizable principles applicable to novel patterns.

The provided example demonstrates this distinction:

```
Example: Tool selection for "check my order #12345"
Input: "check my order #12345"
Selected tool: lookup_order
Reasoning: The user provides an order number (#12345), indicating
they want order-specific information. Even though this could be
interpreted as a general customer query, the specific order
identifier makes lookup_order the correct choice over get_customer.
```

The page emphasizes: "Without the reasoning, the model learns only 'queries mentioning order numbers go to lookup_order.' With the reasoning, the model learns the general principle: specific identifiers route to specific lookup tools."

**Coverage Rule: Address Failing Scenarios**
Examples must directly target documented failure patterns. If extraction works on tables but fails on narrative text, examples must demonstrate correct extraction from narrative structures. If code review is inconsistent on variable shadowing, examples must show variable shadowing classifications at different severity levels.

### Hallucination Reduction Effect

Few-shot examples produce a beneficial side effect: they "cut hallucination in extraction tasks." When models see examples of correct extraction from varied structures—inline citations vs. bibliographies, narrative descriptions vs. structured tables, headers vs. embedded text—they "learn to handle structural variety without inventing data."

The page illustrates a practical scenario: "A financial report might list expenses in a table on one page and bury them in a paragraph on the next. Without examples, the model often nails the table but returns empty fields for the narrative section, or worse, fabricates values. Show it both structures and extraction quality climbs."

### Few-Shot for Reducing False Positives

In code review contexts, few-shot examples serve dual purposes: they demonstrate both what to flag and what to ignore. A detailed example shows variable shadowing assessment:

```
Code: function process(items) {
  const result = items.map(item => {
    const result = transform(item);  // shadows outer 'result'
    return result;
  });
  return result;
}
Severity: minor
Reasoning: The inner 'result' shadows the outer variable but
within a limited scope (arrow function). The code is still readable
and the shadow does not cause a bug. This is a style preference,
not a defect. Flag as minor only if style consistency is in scope.
```

This teaches the model "to distinguish genuine bugs from benign patterns, reducing false positives while preserving the ability to generalise to genuinely problematic shadowing cases."

### Few-Shot vs. Tool Granularity — Two Distinct Concepts

These are separate levers that solve different problems and should not be confused:

- **Few-shot prompting** is about *examples embedded in the prompt content* to drive output consistency and resolve ambiguous judgement calls. It operates at the level of "how does the model interpret and format its response given this input." It fixes: inconsistent formatting, inconsistent judgement on ambiguous cases, and empty/hallucinated extraction fields.
- **Tool granularity** (covered under tool-use/agent design elsewhere in the guide, not this page's main focus) is about *how many tools an agent has access to and how narrowly each tool's scope is defined* — an architectural/API-surface decision about the agent's action space, not about examples in a prompt.

The diagnostic table below reflects this distinction implicitly: few-shot examples are the fix for inconsistent output/judgement/extraction problems, while "wrong tool selection" is first addressed via better tool descriptions (a tool-definition/granularity concern), with few-shot examples only as a secondary reinforcement once tool descriptions are already clear. In other words: if the problem is "the model chose the wrong tool," first ask whether the tool's description/scope is ambiguous (a tool-granularity/definition problem); only after that is fixed does few-shot prompting apply to further disambiguate genuinely hard edge cases.

### Few-Shot vs. Alternative Techniques: Diagnostic Table

The page provides a comparison matrix distinguishing when few-shot examples apply versus other techniques:

| Problem | Correct Technique |
|---------|-------------------|
| Inconsistent output formatting | Few-shot examples |
| Malformed JSON output | tool_use with JSON schemas |
| Fabricated values for missing fields | Optional/nullable schema fields |
| Wrong tool selection | Better tool descriptions (first), then few-shot |
| Model misses information in narrative text | Few-shot examples showing narrative extraction |
| Extraction sum does not match total | Validation-retry loop |

### Exam Traps: Explicitly Identified Misconceptions

**Trap 1: Choosing 'add more detailed instructions'**
"If detailed instructions already exist and output is still inconsistent, adding more instructions will not fix the problem. Few-shot examples demonstrating the exact desired format are more effective for consistency."

**Trap 2: Literal Pattern-Matching Misunderstanding**
"Thinking few-shot examples only teach literal pattern-matching" misses that "When examples include reasoning for why decisions were made, they teach the model to generalise to novel patterns. The model learns the decision principle, not just the specific case."

**Trap 3: Misapplying Confidence Thresholds**
"Using confidence thresholds to fix inconsistent judgement calls" is incorrect because "Confidence thresholds are poorly calibrated and do not address the root cause. Few-shot examples showing the correct judgement for ambiguous cases directly teach consistent decision-making."

### Practice Scenario

A provided scenario presents an extraction pipeline correctly handling structured tables but returning empty fields for identical information in narrative paragraphs, with detailed instructions already specifying all required fields. Four options are presented:

- **Option A**: Pre-processing narrative text into structured tables
- **Option B**: Increasing model context window
- **Option C**: Adding few-shot examples showing extraction from both tables and narratives (correct answer)
- **Option D**: Post-processing retry for empty fields

The correct answer is Option C, demonstrating that few-shot examples addressing the specific failing scenarios (narrative extraction) represent the first-line intervention.

### Build Exercise Requirements

The hands-on exercise requires:

1. Creating a baseline extraction prompt without examples against 10 documents with varied structures to establish the consistency problem
2. Recording which specific fields are inconsistently empty or incorrect across document types
3. Creating 3 few-shot examples with reasoning for each, targeting documented failure patterns
4. Re-running the same 10 documents and quantifying improvements in empty field rates, format consistency, and extraction accuracy
5. Documenting which structural patterns benefit from few-shot examples and which require alternative techniques (schema changes, validation loops, etc.)

The exercise emphasizes that "Detailed instructions alone produce inconsistent output across varied document structures, which is the exact trigger for deploying few-shot examples."

### Key Concept Synthesis

The page's concluding key concept states: "Few-shot examples are the most effective technique for consistency. Use 2-4 targeted examples that include reasoning for decisions, not just input-output pairs. Deploy them when instructions alone produce inconsistent results, ambiguous judgements, or empty extraction fields for data that exists."

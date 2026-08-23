# Tool Interface Design

## Source
https://claudecertificationguide.com/learn/2-tool-design-mcp/2-1-tool-schema-design

## Summary

### Core Principle
Tool descriptions function as "the PRIMARY mechanism LLMs use for tool selection," not supplementary metadata. This is the foundational concept tested throughout the exam.

### Five Essential Elements of Production-Grade Tool Descriptions
Every tool description must contain:

1. **Purpose Statement** — unambiguous primary function
2. **Input Specification** — data types, formats, constraints, required vs. optional fields
3. **Example Queries** — concrete use cases anchoring the model's understanding
4. **Edge Cases & Limitations** — what the tool does NOT do and behavior outside expected ranges
5. **Explicit Boundaries** — clarification when to use THIS tool versus similar alternatives

### Minimal vs. Production-Grade Descriptions: Concrete Example

**Problematic minimal approach:**
- `get_customer`: "Retrieves customer information"
- `lookup_order`: "Retrieves order details"

**Correct production-grade approach:**
- `get_customer`: "Looks up a customer account by email address, phone number, or customer ID. Returns customer profile (name, contact details, account status, loyalty tier). Use this when you need to verify who the customer is. Do NOT use for order-specific queries — use lookup_order for those."
- `lookup_order`: "Retrieves order details by order number (format: #NNNNN) or tracking ID. Returns order status, items, shipping details, and refund eligibility. Use this when a customer asks about a specific order. Do NOT use for customer identity verification — use get_customer for that."

The second version provides explicit disambiguation including accepted identifier formats and clear boundary statements.

### The Misrouting Problem
Overlapping or nearly identical descriptions cause selection confusion. Example: when `get_customer` and `lookup_order` have minimal descriptions, an agent frequently routes "check my order #12345" to `get_customer` instead of `lookup_order`.

#### Correct vs. Incorrect Remedies
The exam tests prioritization of solutions. When misrouting stems from weak descriptions with a workable number of tools:

**Correct first-step remedy:**
- Expand descriptions with all five elements — low effort, high leverage, directly addresses root cause

**Incorrect approaches (in priority order of how tempting/wrong they are):**
1. **Few-shot examples** — adds token overhead without fixing the core confusion; treats symptoms rather than disease
2. **Routing classifier** — over-engineered, bypasses LLM natural language understanding, adds unnecessary infrastructure complexity
3. **Tool consolidation** — valid long-term architecture but requires significantly more effort than description expansion

The material emphasizes: "The exam consistently favours low-effort, high-leverage fixes. Better descriptions before routing classifiers."

### Critical Caveat: When Descriptions Alone Aren't Sufficient
The material explicitly states this crucial condition: descriptions fix misrouting "when the agent has a workable number of tools and simply cannot tell two of them apart. They are **not** the fix when the toolkit itself is the problem."

The threshold mentioned: "past roughly 4-5 tools per agent, selection degrades on decision complexity alone, and rewriting 22 descriptions leaves that untouched."

**Diagnosis requirement:** "Diagnose which one you are looking at before reaching for a remedy. Task Statement 2.3 covers the overload threshold and what to do instead."

### Tool Splitting Strategy
Generic tools with broad responsibilities create ambiguity.

**Before splitting:**
- `analyze_document`: "Analyses a document and returns results"

**After splitting:**
- `extract_data_points`: "Extracts structured data fields (dates, amounts, names) from a document"
- `summarize_content`: "Produces a concise summary of a document's key arguments and conclusions"
- `verify_claim_against_source`: "Checks whether a specific claim is supported by the source document, returning supporting/contradicting evidence"

This creates purpose-specific tools with defined input/output contracts. The model can select the correct tool based on actual user needs.

### Tool Renaming for Clarity
Confusingly similar tool names create overlap at the interface level. Example: rename `analyze_content` to `extract_web_results` with a web-specific description. This eliminates ambiguity without implementation changes.

### System Prompt Interactions — Subtle Failure Mode
Keyword-sensitive instructions in system prompts can create unintended tool associations that override well-written descriptions. Example: "If your system prompt says 'always check customer details before proceeding', the model may route any customer-related query to `get_customer` no matter what the descriptions say."

**Required action:** After updating tool descriptions, reread the system prompt for conflicts. "It's a subtle failure mode, and the exam tests it."

### Identified Exam Traps
1. **Few-shot examples for misrouting** — add token overhead without addressing root cause; fix descriptions first
2. **Routing classifier as first step** — over-engineered and adds disproportionate infrastructure complexity
3. **Tool consolidation as first step** — requires more effort than expanding descriptions; not proportionate as an initial response
4. **Ignoring system prompt conflicts** — keyword-sensitive instructions can silently override improved tool descriptions, creating unintended associations

### Practice Scenario
Production logs show agents frequently call `get_customer` when users ask about orders ("check my order #12345") instead of calling `lookup_order`. Both tools have minimal descriptions accepting similar identifier formats.

**Correct answer:** Expand each tool description to include input formats, example queries, edge cases, and boundaries explaining when to use it versus similar tools.

**Why other options fail:**
- Few-shot examples add overhead without fixing confusion
- Consolidation requires excessive effort
- Routing layer bypasses natural language understanding

### Build Exercise Requirements
1. Create two MCP tools with intentionally ambiguous descriptions to reproduce misrouting
2. Test with 10 queries covering different intents, logging tool selection for each
3. Rewrite descriptions to include: purpose, expected inputs with formats, example queries, edge cases, explicit boundaries (each 3-5 sentences, including statements like "Do NOT use for X — use Y instead")
4. Re-run the same queries to compare selection accuracy before/after (expecting 9/10 or 10/10 correct post-improvement)
5. Review the system prompt for keyword-sensitive instructions that could override improved descriptions

### Key Learning Objectives
- Understanding that tool descriptions are the primary selection mechanism
- Writing production-grade descriptions with all five elements
- Diagnosing misrouting caused by ambiguous/overlapping descriptions
- Identifying system prompt conflicts overriding well-written descriptions

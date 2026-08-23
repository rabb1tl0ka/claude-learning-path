# Tool Distribution & Tool Choice

## Source
https://claudecertificationguide.com/learn/2-tool-design-mcp/2-3-tool-distribution-choice

## Summary

### Core Problem Statement
"The number of tools you give an agent directly affects how reliably it selects the right one." This is characterized as an architectural decision, not merely an implementation detail.

### The Tool Overload Problem
**Optimal range:** 4-5 tools per agent, scoped to that agent's specific role.

**Core principle:** "Each agent gets only the tools it needs for its defined role. Nothing more."

**The relevance issue:** Agents misuse tools outside their specialization. Example given: a synthesis agent with access to `web_search` might run its own searches instead of using already-provided results, duplicating work and wasting context.

### Consolidating Near-Duplicate Tools
When multiple tools perform the same type of operation (data in, operation, data out), they should be consolidated into a single parameterized tool with an enum parameter rather than split across multiple tools.

**Example:** 22 tools (3 query tools, 19 transformations like `pivot_table`, `calculate_percentile`, `normalise_currency`) should become 4 tools by consolidating the 19 transformations into a single `transform_data` tool with a `transform_type` enum parameter.

**Reasoning:** "Selection accuracy improves because the hard choice got smaller, not because the capability did."

#### Decision Matrix for Tool Remedies

| Symptom | Fix |
|---------|-----|
| Few tools, two read alike | Sharpen descriptions (Task 2.1) |
| Different jobs (query, transform, export) | Split by role, 4-5 tools each |
| Variations on one job, sharing a shape | Consolidate into one parameterized tool |
| Doing more than agent should do | Constrain them |

**Critical warning:** "Count the tools before you pick the remedy." The same misrouting symptom from 5 tools is a description problem; from 22 tools it is a decision complexity problem requiring consolidation.

**Server boundary misconception:** Moving tools to a second MCP server does not reduce the problem. "Server boundaries are invisible to the model. A client hands it every tool from every connected server as one flat list."

### The tool_choice Parameter
Three settings with distinct functions:

#### 1. "auto" (Default)
```json
{
  "tool_choice": { "type": "auto" }
}
```
"The model decides whether to call a tool or return text. Use this for general operation where the model needs flexibility to respond conversationally when no tool call is appropriate."

#### 2. "any"
```json
{
  "tool_choice": { "type": "any" }
}
```
"The model MUST call a tool but chooses which one. Use this when you need guaranteed structured output from one of multiple schemas — the model will always produce a tool call, never plain text."

**Primary use case:** "Extraction pipelines are where this earns its keep. If you have multiple extraction schemas (one for invoices, one for receipts, one for contracts) and the document type is unknown, `"any"` guarantees the model picks one and produces structured output rather than returning a conversational response."

#### 3. Forced Selection
```json
{
  "tool_choice": { "type": "tool", "name": "extract_metadata" }
}
```
"The model MUST call a specific named tool. Use this to enforce mandatory first steps — the model cannot skip or reorder the required operation."

**Use case:** "If metadata extraction must happen before any enrichment tools run, forced selection guarantees it. The model can't decide to skip `extract_metadata` and jump straight to enrichment. After the forced call completes, subsequent turns can use `"auto"` for the remaining steps."

### Scoped Cross-Role Tools
**Problem:** Routing every cross-role request through the coordinator adds "2-3 round trips per request and can increase latency by 40% or more."

**Solution:** A **scoped cross-role tool** — a constrained version of the capability, given directly to the agent that needs it.

**Example scenario:** A synthesis agent needs frequent fact verification during report generation. Analysis shows "85% of verifications are simple lookups that take milliseconds." Instead of routing all through the coordinator, give the synthesis agent a scoped `verify_fact` tool for simple lookups while routing complex verifications (requiring multiple sources, cross-referencing, or real judgment) through the coordinator.

**Benefit:** "The 85% simple case is handled locally; the 15% complex case uses the full pipeline."

### Least Privilege in Tool Design
**Generic tool problem:** Giving a subagent `fetch_url` allows fetching anything from anywhere.

**Constrained alternative:** Give `load_document` that validates document URLs only.

**Advantages:**
- Prevents misuse (agent cannot fetch arbitrary URLs)
- Clarifies tool purpose (specific, not generic description)
- Reduces risk of unintended side effects (no non-document resources)

This applies "least privilege applied to tool design. Each tool does exactly what the agent needs and nothing more."

### Multi-Agent Research System Example

| Agent | Tools (4-5 each) |
|-------|------------------|
| Web Search | `search_web`, `fetch_page`, `extract_links`, `save_snippet` |
| Document Analysis | `extract_metadata`, `extract_data_points`, `summarize_content`, `verify_claim` |
| Synthesis | `compile_report`, `verify_fact` (scoped), `format_citation`, `assess_coverage` |
| Coordinator | `Agent` (formerly `Task`), `review_output`, `request_revision` |

**Design principle:** "Each agent has exactly the tools it needs. The synthesis agent has a scoped `verify_fact` for simple lookups. The coordinator runs the workflow without holding any domain-specific tools itself."

### Exam Traps
1. **Round-trip latency:** Routing all simple verification through coordinator when 85% are simple lookups adds unnecessary hops. Scoped tools cut latency up to 40%.
2. **tool_choice 'auto' with structured output required:** Model may return conversational text instead of calling a tool. Use `"any"` to guarantee a tool call, or forced selection for a specific tool.
3. **18-tool agent:** Expecting reliable selection from 18 tools is fundamentally flawed. Optimal range is 4-5 tools per agent.
4. **Generic tools on subagents:** Giving a subagent `fetch_url` instead of constrained `load_document` enables misuse and violates the least privilege principle.

### Practice Scenario
**Situation:** Synthesis agent frequently returns to coordinator for fact verification, adding 2-3 round trips per task and 40% latency. Analysis shows 85% of verifications are simple lookups.

**Correct answer:** Option A — "Give the synthesis agent a scoped verify_fact tool for simple lookups, routing only complex verifications through the coordinator."

This directly addresses the identified problem using the scoped cross-role tool pattern.

### Build Exercise Learning Objectives
1. Scope tools to agent roles using the 4-5 tools per agent guideline
2. Implement scoped cross-role tools to avoid coordinator round-trip latency
3. Configure tool_choice modes (auto, any, forced) for different workflow requirements
4. Apply least-privilege tool design by replacing generic tools with constrained alternatives
5. Verify tool distribution prevents cross-role misuse in multi-agent systems

# 5.3 Error Propagation in Multi-Agent Systems

## Source
https://claudecertificationguide.com/learn/5-context-management/5-3-error-propagation

## Summary

### Page Structure & Navigation Context

Lesson 5.3 in Domain 5 (Context Management & Reliability) of the Claude Certified Architect Foundations (CCAR-F) exam curriculum. Includes interactive learning tools, a practice scenario with multiple-choice answers, and a 50-minute build exercise with five implementation tasks.

### Core Concept: Error Propagation's Role in System Reliability

"Error propagation determines whether a multi-agent system recovers gracefully or fails silently." When a subagent encounters failure — timeout, permission error, invalid query — the mechanism by which failure information returns to the coordinator dictates overall system reliability. The exam tests three knowledge areas: structured error context, two critical anti-patterns, and the distinction between access failures versus valid empty results.

### Structured Error Context — Four Required Elements

When a subagent fails, it must return structured error context enabling intelligent coordinator recovery decisions:

1. **Failure Type Categorization** — one of four types, each requiring a different recovery strategy:
   - **Transient failures:** timeouts and rate limits. May succeed on retry.
   - **Validation failures:** bad input or malformed queries. Require fixing the query before retry.
   - **Business failures:** rule violations or policy constraints. Demand escalation or an alternative approach.
   - **Permission failures:** access denied or authorization issues. Cannot be retried without authorization changes.

2. **What Was Attempted** — the specific query, parameters used, and target system. Actionable context: "Searched academic database for 'renewable energy policy' with date range 2022-2024" rather than "Search failed."

3. **Partial Results Gathered Before Failure** — if the subagent retrieved some but not all results before failure (e.g. three of five sources before timeout), those partial results are valuable and must be preserved. Discarding partial results because the overall operation failed is wasteful.

4. **Potential Alternative Approaches** — the subagent, knowing its domain, should suggest recovery strategies: trying a different database, broadening search terms, or checking cached results. These help the coordinator decide recovery strategy.

Example structured error response:
```json
{
  "status": "partial_failure",
  "failureType": "transient",
  "attemptedAction": {
    "tool": "search_academic_db",
    "query": "renewable energy policy",
    "dateRange": "2022-2024"
  },
  "partialResults": [
    {
      "title": "EU Renewable Energy Directive 2023",
      "source": "EUR-Lex",
      "retrieved": true
    }
  ],
  "alternativeApproaches": [
    "Retry with narrower date range (2023-2024)",
    "Search alternative database: government_publications",
    "Use cached results from previous research session"
  ]
}
```

"This structure gives the coordinator everything it needs to decide: retry the same query, try an alternative, proceed with partial results, or escalate."

### The Two Critical Anti-Patterns

**Anti-Pattern 1: Silent Suppression.** Returning empty results marked as success when failure actually occurred. Mechanism: a subagent encounters a timeout but returns `{ "results": [], "status": "success" }`. The coordinator believes the search executed successfully and found nothing. Why this is the worst anti-pattern: the coordinator will not retry, will not attempt alternatives, and produces a synthesis that silently omits an entire research area. "The output looks correct — it just has gaps that nobody can detect." Real-world consequence: in customer support, the agent might report "no orders found" when the order lookup system was actually down, leading the agent to tell the customer they have no account.

**Anti-Pattern 2: Workflow Termination.** Killing the entire pipeline when a single subagent fails. Mechanism: one subagent times out, and the entire research pipeline crashes. Other subagents that completed successfully have their results discarded. "A disproportionate response that wastes completed work and provides no recovery path."

**The Correct Middle Ground:** Structured error propagation — "the failing subagent reports what happened, the coordinator assesses the damage, and the system continues with partial results or targeted recovery."

### Access Failure vs Valid Empty Result — Critical Distinction

This distinction is explicitly flagged as critical and directly tested on the exam.

**Access Failure:** The tool could not reach the data source. Characteristics: timeout, connection error, permission denial. The search did not execute. Recovery: consider retry with same or modified parameters.
```python
{
    "status": "error",
    "failureType": "transient",
    "message": "Connection timeout after 30s",
    "shouldRetry": True
}
```

**Valid Empty Result:** The tool reached the source and executed the query successfully. It found no matches. "This IS the answer. No retry is needed because the system worked correctly — there simply are no results for this query."
```python
{
    "status": "success",
    "results": [],
    "message": "Query executed successfully. No matching records found.",
    "shouldRetry": False
}
```

**Consequences of Conflation:** Treating access failures as valid empty results means you never retry when you should. Treating valid empty results as access failures means you waste time retrying a query that will always return nothing.

### Coverage Annotations for Transparency

When synthesis agents combine findings from multiple subagents, output should note which topic areas are well-supported versus which have gaps. Principle: if one subagent failed to retrieve sources on a specific topic (e.g. geothermal energy), the synthesis should explicitly state this: "Section on geothermal energy is limited due to unavailable journal access during research." Rationale: "This is far better than silently omitting the topic. Coverage annotations let the consumer know what the report covers fully and where there are known limitations. Without them, a gap in the synthesis looks like the topic was not relevant rather than the source being unavailable."

### Local Recovery for Transient Failures

Subagents should implement local recovery for transient failures before propagating errors to the coordinator. Local recovery mechanisms: retry logic, fallback sources, degraded responses. Rule: only propagate errors the subagent cannot resolve locally. When propagating, always include what was attempted and any partial results gathered. Benefit: "This reduces coordinator complexity. The coordinator doesn't need to manage retry logic for every possible transient failure across every subagent. Each subagent handles its own transient failures and escalates only the persistent ones."

### Key Concept Summary (Official)

"Structured error context (failure type, attempted action, partial results, alternatives) enables intelligent coordinator recovery. The two anti-patterns are silent suppression (empty results as success) and workflow termination (killing the pipeline on one failure). Access failures need retry consideration; valid empty results do not."

### Exam Traps

1. **Timeout to Empty Success** — catching a timeout and returning empty results marked successful. Silent suppression prevents all recovery; the coordinator believes the search succeeded and found nothing, so it will never attempt alternatives. "This is the worst anti-pattern."
2. **Pipeline Termination** — terminating the entire research pipeline when one subagent times out. Wastes partial results from other subagents that completed successfully. The coordinator should assess the failure and decide on targeted recovery.
3. **Generic Error After Retry Exhaustion** — returning a generic "search unavailable" status after retry exhaustion. Generic errors hide the query, partial results, and alternative approaches from the coordinator. Structured error context enables informed recovery; generic statuses prevent it.
4. **Retrying Valid Empty Results** — retrying a valid empty result because it looks like a failure. A valid empty result means the query executed successfully and found no matches — this IS the answer. Retrying wastes time and resources on a query that will always return nothing.

### Practice Scenario & Answer

A web search subagent in a multi-agent research system times out while researching a complex topic. Design how this failure information flows to the coordinator to enable intelligent recovery.

- A (Incorrect): Catch the timeout and return empty result set marked successful — silent suppression.
- B (Incorrect): Propagate exception to top-level handler that terminates entire workflow — workflow termination.
- C (Correct): Return structured error context including failure type, attempted query, partial results, and potential alternative approaches.
- D (Incorrect): Implement automatic retry with exponential backoff, returning generic status only after retries exhausted — hides context from coordinator.

**Correct Answer: C**, which embodies all four elements of structured error context.

### Build Exercise — Five Implementation Tasks

1. **Define Structured Error Schema:** fields — failureType (enum of transient/validation/business/permission), attemptedAction (tool, query, parameters), partialResults (array), alternativeApproaches (string array). Rationale: enables intelligent coordinator recovery; generic error messages prevent all informed recovery. Expected output: TypeScript interface or JSON schema with typed fields and descriptions.

2. **Distinguish Access Failures from Valid Empty Results:** implement subagent distinguishing timeout/connection errors from successful queries returning no matches. Rationale: conflating these is "a critical error the exam tests directly." Expected output: subagent function catching exceptions as access failures (shouldRetry: true) while successful empty-result queries report success with empty results array (shouldRetry: false).

3. **Local Retry Logic with Exponential Backoff:** build retry mechanism (3 retries, exponential backoff — example timing given: 1s, 2s, 4s) within subagent before propagating to coordinator. Rationale: reduces coordinator complexity by handling transient failures locally. Partial results must be preserved across retries.

4. **Coordinator Recovery Decision Logic:** coordinator receiving structured errors and deciding between retry with modified query, alternative approach, or proceed with partial results. Should handle all four failure types differently and never silently suppress errors — "the correct middle ground between silent suppression (ignoring failures) and workflow termination (killing the pipeline on one failure)."

5. **Coverage Annotations for Synthesis Output:** add coverage annotations noting which findings are well-supported versus topic areas with gaps due to unavailable sources. Synthesis output includes a coverage section listing each topic area with data quality status: well-supported, limited (with reason), or unavailable (with reason). Failed subagent topics explicitly noted, not silently omitted.

### Sources Cited

1. Claude Certified Architect Foundations Exam Guide — Domain 5, Task Statement 5.3 (Anthropic)
2. Anthropic Multi-Agent Patterns — Anthropic engineering documentation

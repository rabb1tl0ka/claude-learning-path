# Structured Error Responses

## Source
https://claudecertificationguide.com/learn/2-tool-design-mcp/2-2-structured-error-responses

## Summary

### Core Principle
When MCP tools fail, structured error responses determine whether agents recover intelligently or fail blindly. The MCP protocol provides the `isError` flag to communicate tool failures, signaling execution failure so the model can reason about recovery rather than treating error text as a successful result.

### The Four Error Categories

#### 1. Transient Errors
**Definition:** Timeouts, service unavailability, rate limits — the underlying system is temporarily unreachable but the request itself is valid.

**Recovery strategy:** Retry after a brief delay.

**Example response structure:**
```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Service temporarily unavailable"
  }],
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "The order database is experiencing high load. The request is valid and should succeed on retry."
}
```

#### 2. Validation Errors
**Definition:** Invalid input format, missing required fields, out-of-range values — the request itself is malformed.

**Recovery strategy:** Fix the input and retry.

**Example response structure:**
```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Invalid order ID format"
  }],
  "errorCategory": "validation",
  "isRetryable": true,
  "description": "Order ID must be in format #NNNNN (e.g. #12345). Received: 'order-abc'. Please reformat and retry."
}
```

#### 3. Business Errors
**Definition:** Policy violations, limit exceedances, business rule conflicts — the request is technically valid but violates a business constraint.

**Recovery strategy:** Do NOT retry. The same request will always fail. The agent needs an alternative workflow.

**Critical characteristic:** `isRetryable: false` because business errors never resolve through retrying; the same policy violation applies every time.

**Example response structure:**
```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Refund exceeds policy limit"
  }],
  "errorCategory": "business",
  "isRetryable": false,
  "description": "Refund amount of £750 exceeds the £500 automatic refund limit. This requires manager approval. Please escalate to a human agent with the refund details."
}
```

#### 4. Permission Errors
**Definition:** Access denied, insufficient credentials, authorisation failures — the tool cannot execute because the caller lacks required permissions.

**Recovery strategy:** Escalate or use different credentials.

**Critical characteristic:** `isRetryable: false` because a retry with the same credentials will fail identically.

**Example response structure:**
```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Access denied"
  }],
  "errorCategory": "permission",
  "isRetryable": false,
  "description": "The current service account does not have permission to access financial records. Escalate to a senior agent with financial system access."
}
```

### The isRetryable Flag
The `isRetryable` field answers whether any path to success exists through retrying. It does not promise the same request will succeed unchanged. Two categories share `isRetryable: true` but require different recoveries:

- **Transient errors:** Retry as-is once the system recovers
- **Validation errors:** Retry only after the agent fixes the input (e.g., reformatting `order-abc` to `#12345`)

Two categories share `isRetryable: false`:

- **Business errors:** A business rule blocks the request outright
- **Permission errors:** Need a different principal with the right access, not a reworded call

The guidance is to read the flag as "can a retry ever work," then read `errorCategory` to know how: resend, self-correct, escalate, or take an alternative route.

### Critical Distinction: Access Failure vs. Valid Empty Result
This is "the distinction to nail" and the exam tests it directly.

**Access failure:** The tool couldn't reach the data source. A timeout occurred, authentication failed, or the service was down. The data might exist, but the tool couldn't check. The agent needs to decide whether to retry.

**Valid empty result:** The tool successfully queried the data source and found no matches. The query executed correctly — there simply is no data matching the criteria. The agent should NOT retry. The answer is "no results found."

**The anti-pattern:** "A tool returns an empty array after a customer lookup. The agent retries 3 times, then escalates to a human. Analysis reveals the customer's account simply does not exist." The tool succeeded, queried the database, found no matching customer, and correctly returned an empty result. However, because the response doesn't distinguish between "I couldn't reach the database" and "I reached the database and found nothing," the agent treats both as a failure worth retrying.

**Correct structuring:**

Valid empty result (NOT an error):
```json
{
  "isError": false,
  "content": [{
    "type": "text",
    "text": "No customer found matching email 'john@example.com'. The query executed successfully but returned no matches."
  }],
  "resultCount": 0
}
```

Access failure (IS an error):
```json
{
  "isError": true,
  "content": [{
    "type": "text",
    "text": "Could not reach customer database"
  }],
  "errorCategory": "transient",
  "isRetryable": true,
  "description": "Connection to the customer database timed out after 5 seconds. The query did not execute."
}
```

### Error Propagation in Multi-Agent Systems
In multi-agent architectures, error handling follows local recovery with selective propagation:

1. **Subagents implement local recovery for transient failures:** If a web search times out, the search subagent retries before bothering the coordinator.
2. **Only propagate errors that cannot be resolved locally:** If all retries fail, the subagent reports the failure upward.
3. **Include partial results and what was attempted:** The coordinator needs context — "I searched 3 of 5 sources successfully. Sources 4 and 5 timed out. Here are partial results from the 3 successful sources."

This prevents two anti-patterns: silently suppressing errors (returning empty results as success) and terminating entire workflows on a single failure.

### Structured Metadata Requirements
Every error response should include three fields:

1. **errorCategory** (string): one of `transient`, `validation`, `business`, or `permission`
2. **isRetryable** (boolean): whether retry can ever succeed
3. **description** (string): human-readable explanation of the error and suggested recovery

Without these fields, the agent cannot distinguish a transient timeout from a permanent policy violation and cannot make appropriate recovery decisions.

### Key Exam Traps

**Trap 1:** Retrying when a tool returns an empty result from a successful query. An empty result from a successful query means 'no data matches your criteria.' Retrying will produce the same empty result. The agent should accept the result and respond accordingly.

**Trap 2:** Using generic error messages like 'Operation failed' without structured metadata. Without `errorCategory`, `isRetryable`, and a description, the agent cannot distinguish transient failures from business rule violations and cannot make appropriate recovery decisions.

**Trap 3:** Treating business errors as retryable. Business errors (e.g., refund exceeds policy limit) will never resolve through retry. The same policy violation applies every time. The agent must take an alternative path such as escalation.

**Trap 4:** Silently suppressing subagent errors by returning empty results as success. This hides failure information from the coordinator, preventing intelligent recovery. The coordinator cannot distinguish 'found nothing' from 'could not search' and may produce incomplete or inaccurate output.

### Practice Scenario
**Scenario:** A tool returns an empty array after a customer lookup. The agent retries 3 times, then escalates to a human agent. Analysis shows the customer's account simply does not exist. What is the root cause of this wasted effort?

**Correct answer (Option D):** The tool does not distinguish between access failures and valid empty results, so the agent treats no matches as a retriable failure.

**Why other options are incorrect:**
- Option A (retry limit too low): Raising retries to 5 would still waste effort since the customer doesn't exist.
- Option B (never retry): Overly restrictive — retries are appropriate when a tool genuinely experiences access failures.
- Option C (escalation threshold too aggressive): The issue isn't how many retries occur; it's that retries shouldn't happen at all for a successful empty result.

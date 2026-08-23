# Agent SDK Hooks

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-5-agent-sdk-hooks

## Summary

### Core Definition and Purpose
Agent SDK hooks inject deterministic behavior into probabilistic systems by intercepting tool calls and results to enforce business rules and normalize data. They operate at the boundary between the model's decisions and the real world.

### Two Hook Types and Lifecycle Positions

**PostToolUse Hooks:**
- Execute after a tool completes but before the model processes the result
- Transform and normalize tool output data
- The model receives cleaned, consistent data regardless of source tool
- Example: Converting Unix timestamps (1710489600), ISO 8601 dates ("2024-03-15T12:00:00Z"), numeric status codes (200, 404, 500), and string statuses ("active", "cancelled", "pending") into uniform formats

**PreToolUse Hooks:**
- Execute before a tool runs
- Can block, modify, or redirect outgoing tool calls
- Prevent tool execution if policy violations are detected
- Example: Blocking `process_refund` calls exceeding $500 and routing to human escalation

### Critical Distinction
"PostToolUse hooks transform data after execution. PreToolUse hooks enforce policy before execution."

The exam specifically tests whether candidates understand directional differences — using the wrong hook type means either failing to prevent non-compliant actions or unnecessarily blocking completed work.

### Data Normalization with PostToolUse
PostToolUse hooks solve interpretation inconsistency by standardizing heterogeneous formats:
- Unix timestamps → ISO 8601 dates
- Numeric status codes → human-readable strings
- Currency values → consistent decimal format with currency code
- Regional date formats → single standard format

Without normalization, models must interpret mixed formats on every iteration, breeding inconsistency across sessions.

### Policy Enforcement with PreToolUse
Three application scenarios:
1. **Refund threshold enforcement**: Hook intercepts `process_refund` calls. Amounts exceeding $500 are blocked; the tool never executes.
2. **Compliance prerequisite gates**: Hook intercepts `transfer_funds` calls. If required AML checks haven't completed in the current session, the hook blocks execution and returns an error directing the agent to complete AML verification first.
3. **Manager approval workflow**: Hook intercepts `approve_discount` calls for discounts above 20%. Execution pauses and routes to a manager approval queue. Only after approval does the tool execute.

### Decision Framework: Hooks vs. Prompts

| Requirement | Mechanism | Guarantee |
|-------------|-----------|-----------|
| Must be followed 100% of the time | Hooks | Deterministic |
| Preferred but occasional deviation acceptable | Prompts | Probabilistic |

- Financial loss from single failure → use hooks
- Legal/regulatory risk from single failure → use hooks
- Formatting preference or style guideline → prompt-based guidance acceptable

The exam presents prompt-based solutions as distractors for 100% compliance scenarios. The decision isn't whether prompts are "good enough" — it's whether the consequence of single failure justifies deterministic guarantees.

### Side-by-Side Comparison Examples

**International transfers requiring AML checks:**
- Prompt approach: "Always complete AML verification before processing international transfers." Works 95% of the time; 5% failure rate creates regulatory violations.
- Hook approach: PreToolUse hook blocks `transfer_funds` until `aml_check` returns pass. Works 100% of the time; no transfer executes without AML verification.

**Markdown formatting for responses:**
- Prompt approach: "Format all responses using markdown with headers and bullet points." Works most of the time; occasional plain-text responses pose no business risk.
- Hook approach: Unnecessary overhead; formatting preferences don't require deterministic enforcement.

**Refunds above $500 requiring human approval:**
- Prompt approach: "For refunds above $500, escalate to a human agent." Works most of the time; single failure means large refund processed without approval.
- Hook approach: Intercept `process_refund`, check amount, block if above $500, route to human escalation. Works 100% of the time.

### Practical Data Format Chaos Example
A customer support agent uses three tools with conflicting formats:
1. `get_customer`: Unix timestamps and numeric status codes
2. `lookup_order`: ISO 8601 strings and English status strings
3. `check_shipping`: DD/MM/YYYY dates and single-character codes ("S" for shipped, "P" for pending)

Without PostToolUse normalization, the model must interpret three date formats and three status representations every iteration, occasionally confusing day/month order or misinterpreting "P" as "processed" instead of "pending."

With PostToolUse hook normalization: all dates convert to ISO 8601 ("2024-03-15T12:00:00Z") and all status codes convert to human-readable English ("shipped", "pending", "delivered") before the model sees them.

### Explicit Exam Traps
1. **Using PostToolUse hooks to block policy violations**: PostToolUse runs after execution; the non-compliant action has already occurred. Use PreToolUse hooks (pre-execution) to block actions before they happen.
2. **Enhanced prompts as 100% compliance solutions**: Prompts provide probabilistic compliance. For 100% enforcement requirements (financial operations, regulatory compliance, security checks), only hooks provide deterministic guarantees.
3. **Model-side data transformation instead of PostToolUse hooks**: Relying on the model to normalize heterogeneous data formats introduces inconsistency. PostToolUse hooks ensure clean, consistent data reaches the model every time, regardless of tool source.
4. **Confusing hook direction**: PostToolUse runs after execution; PreToolUse runs before. Using the wrong direction means either missing the opportunity to prevent an action or unnecessarily blocking completed work.

### Practice Scenario Answer
**Scenario**: International transfers occasionally process without required AML compliance checks. The compliance team requires 100% enforcement. Current prompt instructions work approximately 95% of the time.

**Correct approach**: Option A — "Implement a PreToolUse hook that blocks the transfer_funds tool from executing until aml_check returns a verified pass result"

This provides deterministic (100%) enforcement at the correct lifecycle point (pre-execution), eliminating the 5% failure rate that creates regulatory violations.

### Build Exercise Learning Objectives
1. Create three tools returning data in different formats (Unix timestamps with numeric codes, ISO 8601 with English strings, DD/MM/YYYY with single-character codes)
2. Implement PostToolUse hook normalizing all dates to ISO 8601 and all statuses to English strings
3. Verify model receives consistent data across all three tool outputs
4. Add PreToolUse hook blocking `process_refund` calls exceeding $500, redirecting to human escalation
5. Add PreToolUse hook blocking `transfer_funds` until `aml_check` returns pass in current session
6. Test both hooks by attempting to trigger blocked operations, verifying prevention before execution

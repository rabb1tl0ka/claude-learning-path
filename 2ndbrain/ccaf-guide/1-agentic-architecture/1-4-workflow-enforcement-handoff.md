# Workflow Enforcement and Handoff

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-4-workflow-enforcement-handoff

## Summary

### Core Distinction: Two Enforcement Approaches

**Prompt-based guidance** involves placing instructions in the system prompt (e.g., "Always verify customer identity before processing refunds"). This approach is probabilistic with a "non-zero failure rate," working in approximately 90-95% of cases. The model may skip steps, reorder them, or interpret instructions loosely.

**Programmatic enforcement** implements code-level checks, prerequisite gates, or hooks that physically prevent downstream tool execution until prerequisites complete. This is deterministic and works "every time" — the model cannot bypass the restriction regardless of its decisions.

### The Exam Decision Rule (Critical for Test Success)
- **Financial operations** (refunds, transfers, payments): requires programmatic enforcement. A single unverified refund to a wrong account constitutes direct financial loss.
- **Security operations** (identity verification, access control): requires programmatic enforcement. A single verification bypass is a security breach.
- **Compliance operations** (AML checks, regulatory requirements): requires programmatic enforcement. A missed compliance check triggers legal penalties.
- **Low-stakes operations** (formatting preferences, output ordering): prompt-based guidance is acceptable since formatting inconsistencies pose no business risk.

Explicit rule: "When the scenario involves money, security, or compliance, the answer is always programmatic enforcement."

### Prerequisite Gates Mechanism
A prerequisite gate is a programmatic check blocking tool execution until a prior condition is satisfied. Example workflow:
1. Agent accesses three tools: `get_customer`, `lookup_order`, and `process_refund`
2. Gate logic checks: has `get_customer` returned a verified customer ID in the current session?
3. If verified: `process_refund` executes normally
4. If not verified: `process_refund` returns error message: "Cannot process refund — customer identity not verified. Please call get_customer first."

The critical distinction: the gate is implemented in code, not prompt language. The model cannot circumvent it by deciding to skip verification steps.

### The 8% Failure Rate Production Example
Real production data shows a customer support agent processes refunds without account ownership verification in 8% of cases, despite clear system prompt instructions stating verification requirements. The system prompt achieves 92% success but fails 8% of the time. The solution is a programmatic prerequisite gate — not an enhanced prompt — which eliminates failures entirely by physically preventing the incorrect execution order.

### Subagent Lifecycle Hooks (SDK-Specific)
The Claude Agent SDK provides specialized lifecycle events:

- **SubagentStart**: fires when a subagent is spawned via the Agent tool (formerly Task). Observational only — receives the subagent's type and ID, can log the spawn or inject additional context. It cannot block or modify the invocation. To enforce rules on spawning itself (rate limits, checking for required context), attach a PreToolUse hook to the Agent tool instead, which can deny or rewrite the outgoing invocation before the subagent starts.
- **SubagentStop**: fires when a subagent finishes and returns results. Receives the subagent's ID and final message, enabling output validation and completion logging. If validation fails, the hook returns `decision: "block"` with a reason, sending the subagent back to continue working rather than finishing. SubagentStop does not transform output; to reshape or redact coordinator-visible results, use a PostToolUse hook on the Agent tool, whose `updatedToolOutput` field replaces the tool result before the model reads it.
- **Subagent-scoped hooks**: Subagents can define their own PreToolUse and PostToolUse hooks in AgentDefinition frontmatter. These intercept only that specific subagent's tool calls, not the coordinator or other subagents. Enables per-subagent policy enforcement (e.g., a billing subagent with refund threshold blocking, a technical support subagent without such restrictions).
- **Stop hook auto-conversion**: When a subagent's frontmatter defines Stop hooks, these automatically convert to SubagentStop events at runtime.

### Multi-Concern Request Handling
When customers submit compound requests (e.g., "return my order, update my shipping address, and ask about loyalty points"), the correct approach involves three steps:
1. **Decompose** the request into distinct items
2. **Investigate each in parallel** using shared context (customer account information relevant to all concerns)
3. **Synthesise a unified resolution** addressing all items in a single response

The incorrect approach is sequential handling with separate conversations or addressing only the first item while forgetting others.

### Structured Handoff Protocol (Critical Constraint)
When an agent escalates to human agents, the fundamental constraint is that human agents "do NOT have access to the conversation transcript." They cannot scroll through chat history to understand context.

A proper handoff summary must be self-contained and include five mandatory fields:
1. **Customer ID** — enabling the human agent to pull up the account
2. **Conversation summary** — what the customer requested and what the agent attempted
3. **Root cause analysis** — the agent's assessment of the underlying issue
4. **Refund amount** (if applicable) — the specific financial figure, not vague references
5. **Recommended action** — what the agent believes the human agent should do

If the summary is incomplete, the human agent must ask the customer to repeat everything, creating poor experience. The summary is the sole information the human agent receives.

### Exam Traps (Explicitly Called Out)
1. **Enhanced system prompts as solutions for high-stakes failures**: "add stronger instructions to the system prompt" and "include few-shot examples showing correct workflow" are distractors for financial, security, and compliance scenarios. These improve accuracy but never eliminate the failure rate: "Enhanced system prompt instructions...improve probability but do not eliminate the failure rate."
2. **Few-shot examples as sufficient for compliance**: Improve model behavior but remain probabilistic. Cannot provide the 100% enforcement required for financial and compliance operations. Only programmatic prerequisite gates achieve deterministic guarantees.
3. **Routing classifiers to fix per-agent compliance issues**: A routing classifier determines which agent handles requests, but compliance failures occur within agent execution sequences, not at routing. Classifiers handle routing, not per-agent workflow enforcement.
4. **Incomplete handoff summaries**: Omitting customer ID or recommended action fails because human agents lack conversation transcript access. All five fields must be populated with specific information.

### Practice Scenario Answer
Production data reveals an 8% failure rate in processing refunds without account ownership verification, despite clear system prompt instructions. Four options:
- **Option A** (few-shot examples): Rejected — probabilistic improvement, not deterministic
- **Option B** (programmatic prerequisite gate): Correct — provides 100% enforcement
- **Option C** (stronger system prompt): Rejected — improves probability from 92% to perhaps 96-97%, never reaches 0%
- **Option D** (routing classifier): Rejected — addresses routing, not workflow enforcement within agent execution

Correct answer: **Option B**.

### Build Exercise Learning Objectives
1. Three tool definitions with proper JSON Schema: `get_customer`, `lookup_order`, `process_refund`
2. Session-level state tracking that records whether `get_customer` has returned verified customer status
3. Verification testing by prompting the agent to skip verification — demonstrating the gate blocks the attempt
4. Structured handoff function producing objects with all five required fields (no empty fields or placeholders)
5. Multi-concern request testing with return, billing dispute, and account update — verifying decomposition, parallel investigation, and unified handoff coverage

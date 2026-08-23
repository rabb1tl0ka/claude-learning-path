# 5.4 Codebase Exploration & Context Degradation

## Source
https://claudecertificationguide.com/learn/5-context-management/5-4-codebase-exploration

## Summary

### Page Structure & Navigation Context

Task 5.4 within Domain 5 (Context Management & Reliability) of the CCAR-F certification curriculum. Includes interactive learning tools, practice scenarios, and a 60-minute build exercise.

### Core Concept: Context Degradation

**Definition & Observable Behavior:** Context degradation occurs during extended codebase exploration when "the model simply loses its grip on earlier findings as the context fills with verbose discovery output." Critical distinction: this represents an **attention quality problem**, not a token limit problem.

**Observable manifestation:** The model transitions from referencing specific discoveries to generic descriptions. Example: instead of stating "the `OrderRepository` class at `src/repos/order.ts` implements the base `Repository<T>` interface with custom caching in the `findById` method," the agent says "this follows the typical repository pattern."

**Root Cause Mechanism:** Accumulation process:
1. Each exploration step generates verbose output (file contents, search results, directory listings).
2. This output accumulates within the conversation context.
3. Earlier precise discoveries become buried deeper in the context window.
4. The model's attention shifts toward recent verbose output.
5. Specific references to earlier findings are lost despite adequate token space remaining.

Critical insight: "Increasing the context window doesn't fix it. The model isn't running out of space."

### Primary Mitigation Strategy: Scratchpad Files

**Implementation & Purpose:** Scratchpad files persist knowledge outside the conversation context, making discoveries "immune to context degradation." The agent writes key findings to a persistent file and references it for subsequent questions rather than relying on conversation history.

**Scratchpad Content Structure** should record:
- **Key Classes:** specific class names with file paths and implementation details (e.g. "OrderRepository (src/repos/order.ts) — implements Repository<T>, custom findById caching").
- **Dependency Chains:** complete call paths (e.g. "RefundProcessor → OrderService → OrderRepository → PostgreSQL; RefundProcessor → PaymentGateway → Stripe API").
- **Critical Findings:** specific issues discovered (e.g. "RefundProcessor has no retry logic for Stripe API failures"; "OrderRepository caches by orderId but cache invalidation on status change is missing").

**Deployment Timing:** "Treat this as a deliberate strategy from the outset, not a rescue move once things degrade — agents should be instructed to maintain scratchpad files from the start of any extended exploration session."

### Secondary Mitigation: Subagent Delegation

**Strategic Purpose:** operates on context isolation, not primarily parallelization. "The real value is context isolation. The main agent's context stays clean for high-level coordination while subagents handle the verbose exploration."

**Delegation Model:** instead of the coordinator agent performing all exploration directly, specific investigation tasks are delegated, e.g.:
- "Find all test files for the order service and report their coverage status"
- "Trace the refund flow from API endpoint to database and list all intermediate services"
- "Identify all external API integrations and their error handling patterns"

**Subagent Context Management:** each subagent operates with isolated context, explores verbosely without polluting the main agent's context, and returns **structured summaries** (not raw verbose output) with key findings. The coordinator retains only essential information while detailed exploration happens separately.

### Tertiary Mitigation: Summary Injection Between Phases

**The Cold Start Problem:** when exploration occurs across phases (e.g. Phase 1: architecture understanding; Phase 2: component investigation), Phase 2 subagents risk duplicating Phase 1 exploration if not provided previous findings — wasted context and re-exploration.

**Solution Implementation:** after Phase 1 completion, summarize key findings and inject these summaries into the initial context of Phase 2 subagents, including:
- High-level architectural patterns (e.g. "The system follows a layered architecture: Controllers → Services → Repositories → Database")
- Critical flow paths (e.g. "The refund flow passes through: RefundController → RefundProcessor → OrderService → PaymentGateway")
- Key concerns identified (e.g. "RefundProcessor has no retry logic for external API failures")
- Specific Phase 2 objectives

This ensures Phase 2 agents have "the architectural understanding needed to ask the right questions" without redundant exploration.

### Quaternary Mitigation: The /compact Command

**Function & Deployment:** Claude Code provides a `/compact` command that "summarises the conversation to free up space while preserving key information" during extended sessions. It condenses verbose discovery output (file contents, search results, directory listings) while maintaining essential findings.

**Critical Usage Pattern:** "Use `/compact` proactively during extended exploration sessions, not just when you hit context limits. It's there to protect context quality, not only quantity." This is preventive context management rather than emergency intervention.

### Crash Recovery: Structured State Manifests

**Problem & Context:** extended exploration sessions can fail through session crashes, network interruptions, or context exhaustion. Without recovery mechanisms, "all exploration progress is lost."

**Manifest Structure & Content:** each agent exports structured state to a known file location (typically JSON) containing:
- **sessionId:** unique identifier (e.g. "explore-order-service-001")
- **phase:** current exploration phase number
- **exploredPaths:** array of file paths already read (e.g. ["src/repos/order.ts", "src/services/order.ts", "src/services/refund.ts"])
- **keyFindings:** nested object with:
  - **architecture:** system design patterns
  - **criticalIssue:** major problems discovered
  - **testCoverage:** coverage percentages by component (e.g. {"OrderService": "87%", "RefundProcessor": "12%"})
- **nextSteps:** array of pending investigations

**Resume Process:** on session resumption, "the coordinator loads this manifest and injects it into agent prompts. The agent picks up where it left off without repeating earlier exploration."

### Exam Traps

1. **Context Window as Solution** — "Increasing the context window to solve context degradation" is incorrect. "Context degradation is not about running out of tokens. It is about the model losing track of specific details as verbose output accumulates. A larger window still fills with verbose output."
2. **Subagent Benefits Misunderstanding** — "Assuming subagent delegation is only about parallelisation" misses the primary value: context isolation — keeping the main agent's context clean while subagents handle verbose exploration.
3. **Restart Without State Persistence** — "Restarting a session to fix context degradation without saving state" results in knowledge loss. Correct approach: "use scratchpad files and state manifests to persist findings before restarting, then inject them into the new session."
4. **Reactive /compact Usage** — "Using /compact only when hitting context limits" is suboptimal. Instead, apply it "proactively during extended sessions to maintain context quality, not just as a last resort when context is exhausted."

### Practice Scenario Analysis

A developer productivity agent explores an unfamiliar codebase and after investigating several modules transitions from specific discovery references (class names, dependency chains) to generic pattern descriptions.

**Correct Answer: Option B** ("Have the agent maintain scratchpad files recording key findings and reference them for subsequent questions") — directly addresses attention quality degradation by externalizing specific findings from the conversation context.

Why other options fail:
- **Option A** (increase context window): misidentifies the problem as token limits rather than attention degradation.
- **Option C** (restart session): loses accumulated knowledge without state persistence.
- **Option D** (pre-load entire codebase): creates massive initial context that itself causes degradation.

### Build Exercise: Context-Resilient Codebase Explorer

Five interconnected components (60-minute exercise):

1. **Coordinator with Subagent Delegation:** create a coordinator agent that spawns subagents for specific tasks rather than conducting all exploration itself. Subagents return structured summaries (key findings, file paths, class names) rather than raw verbose output. Expected outcome: coordinator context remains clean throughout extended exploration.

2. **Scratchpad File Management:** agents write key findings (class names, file paths, dependency chains) to a persistent file and read this file before each subsequent exploration step. The scratchpad should contain specific references, not generic summaries.

3. **Summary Injection Logic:** after Phase 1 exploration, generate a summary capturing high-level architecture, key concerns, and specific Phase 2 investigation targets. Inject this summary into the initial prompts of every Phase 2 subagent — preventing cold-start duplication and ensuring contextual understanding.

4. **Crash Recovery via Manifests:** each agent exports structured state (explored paths, key findings, next steps) to a JSON manifest file. The coordinator loads this manifest on session resumption and injects it into agent prompts to continue from the last checkpoint rather than restarting.

5. **Context Degradation Validation Testing:** run two comparison scenarios — one without scratchpad files (expecting degradation to generic references after 4-5 modules) and one with scratchpad files (expecting maintained specific references throughout). Directly validates that the mitigation prevents the observable degradation symptom.

### Key Concept Summary (As Provided)

"Context degradation is not a token limit problem — it is the model losing grip on specific findings as verbose output accumulates. Scratchpad files persist key discoveries outside the context. Subagent delegation isolates verbose exploration. Crash recovery manifests prevent progress loss across sessions."

### Source Attribution

Material sourced from Claude Certified Architect Foundations Exam Guide (Domain 5, Task 5.4) and Claude Code Documentation for context management and command reference.

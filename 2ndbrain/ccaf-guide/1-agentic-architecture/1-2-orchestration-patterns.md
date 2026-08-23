# Multi-Agent Orchestration

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-2-orchestration-patterns

## Summary

### Core Architecture: Hub-and-Spoke Model
The exam tests one specific pattern for multi-agent systems: hub-and-spoke architecture with two distinct roles:

- **Coordinator Agent (Hub)**: Sits at the center, receives the initial task, decomposes it into subtasks, determines which subagents to invoke, provides context to them, aggregates their results, handles errors, and routes all information between agents.
- **Subagents (Spokes)**: Handle specialized tasks (web search, document analysis, synthesis, report generation). They receive instructions from the coordinator and return results exclusively to it.

### The Cardinal Rule of Communication
"ALL communication flows through the coordinator." Subagents never communicate directly with each other under any circumstances — not for efficiency, not for convenience, not for any reason. The exam simplifies this as an unbreakable rule, though the current Claude Code implementation technically allows nested parent-child delegation. On the exam, direct subagent-to-subagent communication is the wrong answer.

Centralized routing provides three tested benefits:
1. **Observability** — all messages logged and monitored in one location
2. **Consistent error handling** — uniform recovery policies applied by coordinator
3. **Controlled information flow** — coordinator decides what context each subagent receives

### Critical Isolation Principle
Identified as "the single most misunderstood idea in multi-agent systems" — the exam heavily exploits this confusion.

**Subagents do NOT automatically inherit the coordinator's conversation history.** When spawned, subagents receive only what the coordinator explicitly includes in their prompt. They have no access to:
- The coordinator's system prompt (unless explicitly included)
- Previous messages in the coordinator's conversation
- Results from other subagents (unless the coordinator passes them)
- Any shared memory or global state

**Subagents do NOT share memory between invocations.** If the coordinator calls the web search subagent twice, the second invocation has zero knowledge of the first. Every invocation is completely independent.

Practical implication: coordinators must be deliberate about context. Every piece of information a subagent needs must go in its prompt explicitly. If the synthesis agent needs web search results, the coordinator must pass those results — the synthesis agent cannot "look them up" from a shared store because no shared store exists.

### Four Coordinator Responsibilities (Exam-Tested)

1. **Dynamic Subagent Selection**: The coordinator analyzes query requirements and dynamically selects which subagents to invoke. It does NOT always route through the full pipeline. A simple factual question might only need the web search subagent, not the full research-analysis-synthesis chain. Routing every query through every subagent wastes time and resources.

2. **Research Scope Partitioning**: When delegating to multiple subagents, the coordinator partitions the research scope to minimize duplication. It assigns distinct subtopics or source types to each agent — e.g., one agent searches academic papers while another searches news articles; they do not both search the same sources.

3. **Iterative Refinement Loops**: The coordinator evaluates synthesis output for gaps. If synthesis is incomplete, it re-delegates to search and analysis subagents with targeted queries, then re-invokes synthesis until coverage is sufficient. Not a single-shot process — an iterative cycle.

4. **Centralized Communication Routing**: All subagent communication routes through the coordinator for observability, consistent error handling, and controlled information flow.

### The Narrow Decomposition Failure Pattern
A specific failure pattern (referenced as Q7 in sample sets) where a coordinator decomposes "impact of AI on creative industries" into only visual arts subtopics, missing music, writing, and film entirely.

Root cause analysis: **The failure originates in the coordinator's task decomposition, NOT downstream agents.** The web search agent searched thoroughly for what it was assigned. The synthesis agent synthesized everything it received. But the coordinator only assigned visual arts topics, so music, writing, and film were never researched.

The exam explicitly expects you to "trace failures to their origin." When a multi-agent system produces a report missing entire categories, do NOT blame the subagents — check the coordinator's decomposition. This pattern applies broadly: if output is incomplete in scope (not depth), the coordinator's decomposition is almost always the root cause.

#### Practical Example
A multi-agent research system assigned "renewable energy technologies" decomposes this into only "solar panel efficiency" and "wind turbine design." Each subagent produces thorough, well-sourced research. The final report is comprehensive on solar and wind but contains nothing about geothermal, tidal, biomass, or nuclear fusion. The coverage gap exists not because search was poor or synthesis was weak — it exists because the coordinator never assigned those subtopics.

The fix is NOT better search queries, NOT a more capable synthesis agent, and NOT more subagents. The fix is better coordinator decomposition that covers the full breadth of the topic.

### Four Explicit Exam Traps
1. **Blaming downstream subagents for coverage gaps** when the coordinator's task decomposition was too narrow. Subagents research only what they are assigned; failures trace back to the coordinator's decomposition.
2. **Assuming subagents share memory or inherit the coordinator's conversation history.** Subagents have completely isolated context; every piece of information must be explicitly passed in the subagent's prompt.
3. **Proposing direct inter-subagent communication as an efficiency improvement.** Direct communication breaks observability, consistent error handling, and controlled information flow — communication must flow through the coordinator regardless of perceived efficiency gains.
4. **Adding more subagents to fix a decomposition problem.** If the coordinator decomposes a topic too narrowly, adding more subagents doesn't help — they receive equally narrow assignments. Fix the coordinator's decomposition logic.

### Practice Scenario Analysis
**Scenario**: A multi-agent research system produces a report on "renewable energy technologies" covering only solar and wind. Each subagent produced thorough, well-sourced coverage of its assigned topic. The web search subagent returned relevant results for every query. The synthesis subagent accurately combined all research.

**Correct Answer (Option D)**: "The coordinator decomposed the topic into only solar and wind subtopics, never assigning geothermal, tidal, biomass, or fusion to any subagent."

Why other options are wrong:
- Option A: Irrelevant — there was no document analysis subagent described
- Option B: Synthesis cannot request additional research; only the coordinator can assign new tasks
- Option C: The search subagent would return results for whatever queries it received from the coordinator

### Build Exercise Learning Objectives
The build exercise requires constructing a hub-and-spoke research coordinator that:
1. Accepts a broad research topic as input and returns a structured research report, with a system prompt defining its orchestrating hub role
2. Implements task decomposition logic that produces at least 5 distinct subtopics covering full subject breadth (for renewable energy: solar, wind, geothermal, tidal, biomass, and fusion minimum)
3. Spawns two subagents (web search and document analysis) with explicit context passing — all relevant information in each prompt, no reliance on shared state
4. Aggregates results from both subagents and evaluates coverage completeness, with an assessment listing well-covered, partially covered, and missing subtopics
5. Implements an iterative refinement loop that checks coverage, identifies gaps, sends targeted follow-up queries to subagents, and re-evaluates until coverage threshold is met or maximum iteration count is reached
6. Tests with renewable energy technologies and verifies final output covers all six energy types (solar, wind, geothermal, tidal, biomass, fusion) demonstrating 100% completeness

The test case specifically maps to the narrow decomposition failure pattern — if output only covers solar and wind, the root cause is the coordinator decomposition, which is the diagnostic the exam expects.

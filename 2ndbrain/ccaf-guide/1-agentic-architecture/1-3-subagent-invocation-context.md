# Subagent Invocation and Context Passing

## Source
https://claudecertificationguide.com/learn/1-agentic-architecture/1-3-subagent-invocation-context

## Summary

### Core Mechanisms

**The Task Tool (Renamed Agent)**
The coordinator spawns subagents using the Task tool, now called Agent in Claude Code v2.1.63 (June 2026). Task remains functional as an alias. This is not optional — the coordinator's `allowedTools` configuration must explicitly include `"Task"` or `"Agent"` as a binary gate. Without it, the coordinator cannot invoke any subagent regardless of how subagents are defined. The exam expects the answer "Task tool" even though current code emits "Agent" in tool-use blocks.

**AgentDefinition Structure**
Each subagent requires three specifications:
1. Description — explains what the subagent does (guides coordinator's invocation decisions)
2. System prompt — contains the subagent's operational instructions
3. Tool restrictions — scopes which tools the subagent can access, matching its role

### Context Passing: Three Rules

**Rule 1: Complete Findings from Prior Agents**
"If the synthesis subagent needs web search results and document analysis output, the coordinator must pass both — in full — in the synthesis subagent's prompt." Subagents cannot retrieve prior results independently; they operate on isolated context.

**Rule 2: Structured Data Separating Content from Metadata**
Research findings must include both the claim/fact/analysis (content) and source URL, document name, page number (metadata). Passing content without metadata is a documented failure pattern — the synthesis agent produces unsourced claims because it literally has no attribution data to include.

**Rule 3: Goal-Oriented Coordinator Prompts**
Coordinators should specify *what to achieve and quality criteria*, not step-by-step procedures. This enables subagent adaptability; procedural instructions prevent dynamic problem-solving when encountering unexpected situations.

### Structured Metadata Format
The guide provides this exact JSON structure pattern:
```json
{
  "findings": [
    {
      "claim": "Solar panel efficiency has increased 25% in the last decade",
      "source_url": "https://example.com/solar-report",
      "document_name": "Annual Solar Industry Report 2024",
      "page_number": 14,
      "confidence": "high",
      "retrieved_by": "web_search_agent"
    }
  ]
}
```
Each finding carries source attribution as metadata, enabling downstream synthesis agents to produce properly cited reports.

### Parallel Spawning
Independent subagents should be invoked via "multiple Task tool calls in a single response" rather than sequentially across separate turns. Sequential invocation (one subagent per coordinator turn) introduces unnecessary latency. The exam tests latency awareness; correct answers mention "in a single response" or "simultaneously."

### fork_session vs. --resume
`fork_session` creates independent branches from a shared analysis baseline for divergent exploration (e.g., comparing two testing strategies). Forks do not see each other's results, and changes in one fork don't affect others. This differs fundamentally from `--resume`, which continues a specific named session. The exam tests this distinction: fork is for comparing approaches; resume is for continuing the same investigation.

### Exam Traps (Explicit Warnings)
1. **Isolation Misconception**: Assuming subagents automatically access coordinator conversation history or other subagents' outputs. Every required piece of information must be explicitly included in the subagent's prompt.
2. **Attribution Failure Pattern**: Blaming the synthesis agent's prompt for missing citations when the real issue is the coordinator passing content without source URLs and document names. The synthesis agent "can only cite sources it has been given."
3. **Sequential vs. Parallel**: Proposing sequential invocation for independent tasks introduces "unnecessary latency." Independent subagents should spawn in parallel.
4. **fork_session Confusion**: Conflating fork_session with --resume. They serve different purposes and should not be confused.

### Practice Scenario Answer
A synthesis agent produces unsourced claims. Web search and document analysis subagents work correctly. The root cause is **Option B: The coordinator passes content without structured metadata — source URLs, document names, and page numbers are not included**.

The guide explicitly states this is a "specific exam pattern" and warns against selecting options that blame the synthesis agent's prompt (Option C) or propose giving it direct tool access (Option A).

### Build Exercise Focus
The exercise emphasizes: Task/Agent in allowedTools as a hard requirement; defining two research subagents with scoped tool access; designing structured output formats separating content from metadata; passing complete findings with metadata preserved to synthesis agents; verifying attribution for every claim; and refactoring to parallel Task spawning for latency reduction.

# 5.1 Context Window Management

## Source
https://claudecertificationguide.com/learn/5-context-management/5-1-context-window-management

## Summary

### Overview & Core Concept

Context window management is "the foundation of reliable Claude-based systems." Improper context handling produces concrete failures: support agents forgetting refund amounts, research pipelines dropping citations, and extraction systems losing precision on critical fields.

The single most important pattern: the **persistent case facts block** — a structured data container holding transactional facts (amounts, dates, order numbers) that is included in every prompt outside summarized history and never compressed.

### The Progressive Summarization Trap

**The Problem:** When conversations grow long, teams commonly summarize earlier turns to conserve token budget. This strategy systematically destroys critical information in customer-facing and data-processing systems: numerical values, dates, percentages, and customer-stated expectations.

**Example Scenario:**

Original customer message (Turn 3):
```
"I'd like a refund of $247.83 for order #8891 placed on March 3rd"
```

After summarization becomes:
```
"Customer wants a refund for a recent order"
```

The amount, order number, and date — the three facts needed to process the refund — are eliminated. This is not an edge case; it is the default behavior of summarization applied to transactional data.

**The Fix: Persistent Case Facts Block.** Extract transactional facts into a structured block included in every prompt, outside the summarized history. This block never undergoes summarization and persists across every conversation turn.

Example JSON structure:
```json
{
  "caseFactsBlock": {
    "customerId": "C-4421",
    "issues": [
      {
        "orderId": "#8891",
        "orderDate": "2024-03-03",
        "refundAmount": "$247.83",
        "status": "pending_refund",
        "itemDescription": "Wireless headphones — defective"
      }
    ]
  }
}
```

**Multi-issue handling:** For sessions where customers raise multiple problems in one conversation, extract and persist structured issue data into a separate context layer. Each issue receives its own entry with order IDs, amounts, and statuses, preventing cross-contamination during summarization.

### The "Lost in the Middle" Effect

**The Problem:** Models process information at the beginning and end of long inputs reliably. Findings buried in the middle of long context may be missed or given less weight. This is a well-documented phenomenon in LLMs affecting how aggregated inputs are structured.

**The Fix: Structural, Not Prompt-Based.** The solution is architectural rather than instructional. Place key findings summaries at the beginning of aggregated inputs. Organize detailed results with explicit section headers throughout. When feeding a synthesis agent output from multiple research subagents, start with a "Key Findings Summary" section, then provide detailed outputs with clear section boundaries.

Example structure:
```
## Key Findings Summary
- Source A: 12% market growth in renewable sector (2023)
- Source B: Patent filings increased 34% year-on-year
- Source C: Regulatory framework delayed until Q3 2025

## Detailed Findings

### Source A: Market Analysis Report
[Full details here...]

### Source B: Patent Database Analysis
[Full details here...]

### Source C: Regulatory Review
[Full details here...]
```

Key point: Prompt-based reminders to "pay attention to everything" are unreliable for mitigating position effects. Structural placement is required.

### Tool Result Trimming

**The Problem:** Tool results are "a silent context budget killer." An order lookup might return 40+ fields: internal audit timestamps, warehouse codes, shipping carrier IDs, fulfillment centre identifiers, and dozens of other irrelevant fields. For a refund request, only 5 fields are needed, yet those other 35 fields consume tokens in every subsequent turn as conversation history accumulates.

**The Fix: Pre-Context Trimming.** Trim verbose tool outputs to only relevant fields before they accumulate in context. This is not optional ("It is not a nice-to-have").

Example Python implementation:
```python
def trim_order_result(raw_result, relevant_fields=None):
    if relevant_fields is None:
        relevant_fields = [
            "order_id", "order_date", "total_amount",
            "return_eligible", "item_description"
        ]
    return {k: v for k, v in raw_result.items() if k in relevant_fields}
```

**Timing requirement:** Trimming should occur in a `PostToolUse` hook or in the tool implementation itself, before the result enters conversation history. Once verbose data is in context, it persists for every subsequent turn.

### Full Conversation History

**The Stateless API Architecture:** The Claude API is stateless. Each request must include the complete conversation history. Omitting earlier messages causes the model to lose conversational coherence. There is no session state on the server side, so every turn must carry everything the model needs to follow the conversation.

**The Tension and Resolution:** This creates tension with context limits: full history is needed for coherence, but history grows with every turn. The persistent case facts block resolves this by separating critical facts from summarizable narrative, allowing summarization of conversational flow while preserving every transactional detail.

### Upstream Agent Optimization

**The Problem:** In multi-agent systems, upstream agents often return verbose reasoning chains and raw content that downstream agents do not need. When a research subagent sends full thought process to a synthesis agent with limited context budget, the synthesis agent wastes tokens on unusable reasoning.

**The Fix: Structured Upstream Outputs.** Modify upstream agents to return structured data — key facts, citations, relevance scores — instead of verbose content and reasoning chains. Require subagents to include metadata (dates, source locations, methodological context) in structured outputs to support accurate downstream synthesis.

Example JSON structure:
```json
{
  "findings": [
    {
      "claim": "Renewable energy investment grew 12% in 2023",
      "source": "IEA World Energy Report 2024",
      "sourceUrl": "https://example.com/report",
      "relevanceScore": 0.92,
      "publicationDate": "2024-01-15"
    }
  ]
}
```

Benefit: Structured outputs let downstream agents process findings without re-parsing verbose prose. Tokens are conserved, and information fidelity improves.

### Prompt Caching

**The Mechanism:** Prompt caching is the second half of context economics. Instead of trimming what the model sees, you avoid paying to reprocess stable portions. Mark a stable prefix with a `cache_control` breakpoint, and the API stores that processed prefix, reusing it on the next request and charging a fraction of the input cost for cached tokens.

**Layout Determines Hit Rates:** Caching matches from the start of the prompt, prefix by prefix, so layout decides whether you get a hit. Put content that stays constant first: system instructions, tool definitions, long reference documents. Place the `cache_control` breakpoint at the end of that static block. Put volatile content (user's latest message and anything changing per request) after the breakpoint.

Example Python structure:
```python
messages = [
    {
        "role": "system",
        "content": [
            {"type": "text", "text": LONG_STATIC_INSTRUCTIONS},
            {"type": "text", "text": REFERENCE_DOC,
             "cache_control": {"type": "ephemeral"}},
        ],
    },
    {"role": "user", "content": dynamic_user_message},
]
```

**Critical Ordering Rule:** Get the order wrong and you lose the benefit entirely. If dynamic content sits before the static block, the prefix changes on every request, nothing matches, and every call pays full price.

**Cache Lifespan:** The cache is short-lived: an `ephemeral` breakpoint lasts approximately five minutes since last use. Caching pays off for bursts of related requests, not for content reused hours apart.

### Exam Traps

1. **Progressive Summarization Safety** — Misconception: progressive summarization is safe for transactional data. Reality: summarization systematically destroys numerical values, dates, and specific identifiers. A persistent case facts block must hold these outside summarized history.
2. **Prompt-Based Solutions for Position Effects** — Misconception: the "lost in the middle" effect can be solved by instructing the model to pay attention to everything. Reality: the fix is structural — key findings at the beginning of inputs, explicit section headers. Prompt-based reminders are unreliable.
3. **Retaining Full Tool Results** — Misconception: keep full tool results in context because "the model might need them later." Reality: untrimmed tool results from 40+ field lookups exhaust the token budget across turns. Trim to relevant fields before results enter conversation history.
4. **Selective History Truncation** — Misconception: conversation history can be selectively truncated without consequences. Reality: the API is stateless; each request needs complete conversation history. Selective truncation breaks conversational coherence. Use case facts blocks and summarization instead of truncation.

### Practice Scenario

A customer support agent handles a multi-issue session. After several turns, the agent refers to "your recent refund request" instead of the specific $247.83 refund for order #8891. The conversation history is being summarized between turns to manage context length. What is the most effective fix?

- A: Instruct the model to preserve all numerical values verbatim whenever it summarizes the conversation history.
- B: Extract transactional facts (amounts, dates, order numbers) into a persistent case facts block included in every prompt, outside summarized history.
- C: Store the full conversation history in an external database and retrieve relevant turns on demand whenever the agent needs to recall an earlier detail.
- D: Increase the context window size so the full conversation history fits and summarization never needs to run.

**Correct Answer: B** — the persistent case facts block pattern directly addresses the progressive summarization trap by extracting transactional data into a structure that persists outside summarization.

### Build Exercise

Five tasks:
1. **Case facts extractor:** Identify transactional data (amounts, dates, order numbers, statuses) from tool results and return a structured object containing only these facts, excluding non-transactional narrative.
2. **Persistent case facts block implementation:** Create a prompt construction function that always includes the case facts block at the top of every message, followed by summarized history, followed by the current turn, with clear section delimiters.
3. **Tool result trimmer:** Filter order lookup responses from 40+ fields to only the 5 relevant return-related fields, achieving 80-90% size reduction.
4. **Multi-turn conversation validation:** Test a 6-8 turn conversation with summarization after turn 4, verifying that the agent still references exact amounts ($247.83), order numbers (#8891), and dates (March 3rd) from the case facts block.
5. **Key findings placement logic:** Position summaries at the beginning of aggregated inputs with Key Findings Summary sections at the top, followed by detailed results with explicit section headers, to mitigate the lost-in-the-middle effect.

### Source References Cited by the Page

- Claude Certified Architect Foundations Exam Guide — Domain 5, Task Statement 5.1 (Anthropic)
- Anthropic API Documentation — Messages
- Anthropic Prompt Engineering — Long Context Tips

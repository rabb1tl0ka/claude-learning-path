# Batch Processing Strategies

## Source
https://claudecertificationguide.com/learn/4-prompt-engineering/4-5-batch-processing

## Summary

### Core Mechanism: Message Batches API

The Message Batches API is a cost optimization tool with fixed constraints that the exam tests directly.

**Key Financial and Performance Constraints**

- **Cost savings:** 50% reduction compared to synchronous API calls
- **Processing window:** Up to 24 hours maximum (no guaranteed timeframe)
- **Latency SLA:** None guaranteed; results may arrive in minutes or take the full 24 hours
- **Multi-turn tool calling:** Not supported within a single batch request

### The Matching Rule (Most Tested Concept)

This is explicitly identified as "the single most tested concept" from this task statement:

**Synchronous API Use:** Blocking workflows where someone or something awaits results. Examples include:
- Pre-merge checks in CI/CD pipelines
- Real-time code review feedback
- Any workflow where developers are blocked pending completion

**Batch API Use:** Latency-tolerant workflows where results are consumed later. Examples include:
- Overnight technical debt reports
- Weekly code audit summaries
- Nightly test generation runs
- Batch document extraction

The exam specifically presents a scenario (Question 11 in sample questions) where a manager proposes switching everything to batch for cost savings. The correct answer keeps blocking workflows synchronous and only moves latency-tolerant workflows to batch.

### API Implementation Details

**Request Structure**

Each batch request requires:
- **`custom_id` field:** A unique identifier for correlating request/response pairs
- **`params` object containing:**
  - `model`: Specified as "claude-sonnet-5"
  - `max_tokens`: Token limit for response generation
  - `messages` array: Contains the user content

Example provided shows structure mapping documents to requests with sequential custom_id values (e.g., `debt-report-${i}`).

**Failure Handling Pattern**

The correct three-step failure handling pattern is:

**Step 1 - Identify failures by custom_id:**
Parse batch results and filter for errored responses. Code example filters results where `r.result.type === "errored"` and extracts `custom_id` values.

**Step 2 - Resubmit only failures with modifications:**
Do NOT resubmit the entire batch. Common modifications include:
- Chunking oversized documents that exceeded context limits
- Simplifying extraction prompts for documents with unusual structures
- Adding format-specific few-shot examples for documents that failed due to structural variety

Example shows increased `max_tokens` (from 4096 to 8192) and modified custom_id format (`${id}-retry-1`).

**Step 3 - Refine prompts on sample set BEFORE batch processing:**
Test prompts against a representative sample (5-10 documents covering format and edge case ranges) before processing the full batch. This maximizes first-pass success and reduces resubmission costs.

**SLA Calculation Methodology**

The document provides a specific calculation example:

If an organization requires a 30-hour SLA for a report:
- The Batch API guarantees results within 24 hours
- Final batch must be submitted no later than 24 hours before deadline
- Calculation: 30 hours total SLA minus 24 hours processing = 6 hours of buffer
- Submission strategy: Submit batches every 4-6 hours within that buffer window so a fresh batch is always in flight

The exam may present scheduling questions requiring backward calculation from SLA to determine submission frequency.

### Multi-Turn Tool Calling Limitation

The batch API does not support multi-turn tool calling within a single request. This means you cannot:
- Define tools and have the model call them mid-request
- Process tool results and continue the conversation within the same batch item
- Run agentic loops within a single batch request

If a workflow requires tool execution mid-processing, the synchronous API must be used instead. This is identified as "a direct exam test point."

### Prompt Optimization Strategy

The document identifies this as "the most cost-effective batch processing strategy":

1. **Sample set testing:** Take 5-10 representative documents covering format, edge case, and document type ranges
2. **Iterate on the sample:** Refine extraction prompts, add few-shot examples, adjust schema design until achieving high accuracy
3. **Submit the full batch:** With refined prompts, first-pass success rate increases significantly
4. **Handle failures:** Resubmit only failed documents with targeted modifications

Cost impact example: A 90% first-pass success rate on 1,000 documents means 100 retries. A 60% first-pass rate means 400 retries—"four times the resubmission cost, plus the batch processing cost for those retries."

### Exam Traps Explicitly Called Out

**Trap 1: Switching all workflows to batch for cost savings**
Blocking workflows where developers wait for results must remain synchronous. The batch API has no guaranteed latency SLA and can take up to 24 hours. Only latency-tolerant workflows should use batch.

**Trap 2: Assuming batch results arrive quickly**
The batch API has no latency SLA. Results often arrive faster than 24 hours, but blocking workflows cannot be designed around best-case timing. Design around the 24-hour maximum.

**Trap 3: Using batch API for multi-turn tool calling workflows**
The batch API does not support multi-turn tool calling within a single request. If workflow needs tool execution with mid-processing results, the synchronous API is required.

### Practice Scenario

Scenario: A team with two workflows—(1) blocking pre-merge check requiring completion before merge, (2) overnight technical debt report for next-morning review—receives a manager proposal to switch both to Message Batches API for 50% cost savings.

**Correct answer:** Use batch processing for technical debt reports only; keep real-time calls for pre-merge checks.

**Incorrect options provided:**
- Switching both to batch with timeout fallback (misunderstands batch guarantees)
- Keeping real-time for both (ignores cost savings opportunity)
- Switching both with status polling (doesn't address lack of latency SLA)

### Build Exercise Requirements

The exercise requires demonstrating:

1. Classification of 5 workflows as blocking (synchronous) or latency-tolerant (batch-eligible) with justification
2. Batch submission for 20 documents using Message Batches API format with unique custom_id fields
3. Failure handling implementation: parse results, identify failures by custom_id, construct retry batch with increased max_tokens
4. SLA calculation: guarantee 30-hour SLA given 24-hour maximum processing window
5. Sample set refinement: create 5-document sample, iterate prompts, submit full batch of 20 documents

### Terminology and Exact Phrasing

- "Blocking workflows"
- "Latency-tolerant workflows"
- "custom_id fields" (for request-response correlation)
- "The matching rule"
- "24-hour maximum processing window"
- "No latency SLA"
- "First-pass success rate"
- "Resubmit only failures with modifications"

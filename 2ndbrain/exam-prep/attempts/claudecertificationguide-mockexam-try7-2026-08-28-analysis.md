# Try 7 mock exam — failure-pattern analysis

Source: `claudecertificationguide-mockexam-try7-2026-08-28.pdf` (CyberSkill practice exam). A Gemini transcript of Bruno taking the exam out loud ("Claude Learning Path - Try 1" meeting, 2026-08-28) is folded in below — it sharpens the *why* on several misses. Note: the meeting's auto-generated title/notes use Gemini's own session labeling and at one point transcribe Bruno reading the score aloud as "73%" — neither is authoritative; the score below is read directly from the PDF result screen.

**83.33% (50/60) — Pass.** This is the number closest to real CCAF readiness (per `exam-prep/CLAUDE.md`); course/gap mapping below is routing information, not a competing score. Slightly below try 6 (90%), a step down from the recent high point.

## Domain breakdown

| Domain | Score | Note |
|---|---|---|
| code_exploration | 14/16 (88%) | strongest |
| research_pipeline | 12/14 (86%) | |
| extraction_pipeline | 13/15 (87%) | |
| customer_support | 11/15 (73%) | weakest |

## Failure patterns

### 1. Hardcoded rule/threshold picked over trusting an already-capable coordinator or agent to judge (2 misses)

- **Query routing for varying complexity** (research_pipeline): production shows simple fact-checks burn 40+ seconds traversing all four subagents, while complex comparative research genuinely needs the full pipeline, and the query distribution is "diverse and evolving." Picked **a hardcoded fast-path that bypasses subagents entirely for factual questions, full pipeline for everything else**; correct answer was **have the coordinator analyze each query and dynamically decide which subagents to invoke**. The transcript shows this one head-on: Bruno explicitly weighed the dynamic-judgment option ("have the coordinator analyze each query... isn't that patternbased routing or is it a simpler form of that patternbased routing?") and talked himself out of it specifically because the question says complex research "benefits from the full pipeline" — reading that as evidence *for* a rigid two-bucket split, when it's actually just describing one of the many query shapes the coordinator's judgment needs to cover. A fixed fast-path can't handle "diverse and evolving" query shapes; that's exactly the case for judgment, not a rule.
- **Escalation-trigger design** (customer_support): asked to pick the most reliable escalation trigger from four proposals. Picked **escalate after three consecutive failed tool calls**; correct answer was **escalate when the customer requests a human, when the issue requires a policy exception, or when the agent cannot make meaningful progress** — a criteria-based rule, not a call-count threshold. A count-based trigger misses the two most common real escalation reasons (explicit request, policy limits) entirely, and can both over-trigger (three failures on an easy fix) and under-trigger (zero failed calls, but the customer explicitly asked for a human).

Both are the same shape as try 4 Q30 (hardcoded moderation decision tree vs. model-driven judgment) already in the survival guide's "Missing capability vs. already-adequate piece" heuristic — added below as new worked examples rather than a new section.

### 2. MCP tool errors disguised as success or handled by external logic, instead of using the tool result's own error channel (2 misses)

- **`lookup_order` error communication** (customer_support): the backend sometimes returns "Order not found" or a temporary DB failure. Picked **return a success response with a status field indicating the error type**; correct answer was **return the error message in the tool result content with the `isError` flag set to `true`**. The transcript shows real-time doubt about this exact choice — Bruno flagged it himself mid-exam ("temporary database failures returned as a successful response... it's kind of weird... Claude, could you explore this a bit") — the discomfort was right, the answer picked wasn't.
- **`process_refund` error handling** (customer_support): technical errors (transient, 5%) and business errors (permanent, 12%) both currently return a plain-text message, causing wasted retries on errors that can never succeed. Picked **automatic retry logic at the tool level for technical errors only, business errors passed through without retries**; correct answer was **return structured error responses with `retryable: false` for business errors and a customer-friendly explanation**. This still routes the retry decision outside the tool result — the fix that actually generalizes (retryability as structured metadata *on* the tool result) was passed over for control logic bolted on beside it.

Both hit the same underlying mechanism: a tool result already has a dedicated channel for signaling "this call failed and here's why" (`isError`, structured error metadata) — reusing that channel is more reliable than smuggling error info through a success payload or building separate retry/handling logic that has to be kept in sync with it by hand. This is new — not an existing survival-guide section — but it's the same family as the existing "reach for the native tool feature before a generic workaround" shorter note (Edit tool vs. `sed`), generalized from Claude Code's built-in tools to MCP tool design. Added as a new shorter note, flagged draft.

### Other misses (no sibling this attempt)

- **Two branches from one prior analysis** (code_exploration): engineer wants to explore two refactoring approaches in depth from yesterday's findings. Picked **two fresh sessions with a manually-typed summary**; correct was **`fork_session` to branch from yesterday's analysis**. This is a specific-mechanic gap (didn't reach for session forking), not a reasoning pattern — and notably inconsistent within this same exam: a near-identical later question ("compare two testing strategies... resume the analysis session") was answered correctly with `fork_session`. `fork_session` doesn't have a chapter note anywhere in `ccaf-learning/` — flagged as a coverage gap below.
- **Long contract extraction across context limits** (extraction_pipeline): picked **summarize the document first, then extract from the summary**; correct was **chunk with slight overlap, extract per chunk, then merge/reconcile**. Summarizing first is lossy by construction — fields present in the source can be paraphrased away before extraction ever runs. Chunking with overlap is the standard fix, and it's directly covered in `text-chunking-strategies.md` (this was a course-mapped miss, not a gap).
- **Extraction failures from varied source formats** (extraction_pipeline): valid JSON but empty citation/methodology fields, because source documents present that info in inconsistent formats (inline vs. bibliography, section vs. embedded). Picked **a regex-based post-processing layer scanning for citation/methodology patterns**; correct was **few-shot examples demonstrating extraction from documents with varied structures**. Building a parallel regex extractor is new infrastructure duplicating what the model should be doing directly — the actual gap is the model hasn't seen enough format variety in-context, which few-shot fixes at the source.
- **Frustrated customer, no tools called yet** (customer_support): customer demands a human immediately, agent hasn't investigated the account. Picked **call `get_customer` and `lookup_order` first, then escalate**; correct was **acknowledge the frustration and ask one targeted question before escalating**. The transcript shows this was a genuine toss-up between A and D in the moment ("it's either A or D... D first call get customer and look up... then escalate. Oh, that's it") — the tool-investigation instinct won out over engaging with the customer's stated frustration first. Distinct from pattern #1 above: this isn't a hardcoded-rule-vs-judgment question, it's about sequencing empathy before data-gathering when a customer is already upset and hasn't been engaged with at all yet.
- **Edit tool insertion into a repetitive file** (code_exploration): `old_string` can't find unique text in a 150-line file with repetitive docstrings/patterns. Picked **use `replace_all` to target a common pattern and embed the new function in the replacement text**; correct was **Read the file, add the function at the right spot, Write the updated file**. This is the mirror image of the survival guide's existing "reach for `sed`/Bash instead of the Edit tool's native recovery" pattern — here the miss goes the *other* direction: forcing `replace_all` to do a job it's not suited for (bulk-editing every occurrence of a pattern) when the safe move for a single precise insertion is a full Read+Write. Single instance this attempt, logged here rather than folded into the existing heuristic since the direction is opposite.
- **Conflicting metrics from two subagents** (research_pipeline): both report the same metric with moderate confidence but disagree. Picked **include both numbers in the final answer, let the reader decide**; correct was **run a focused check that re-fetches the metric from the primary source and resolves the conflict before synthesizing**. The transcript shows Bruno rejected the verification option outright rather than talking himself out of a correct first instinct — "I don't think that's going to work, it's not going to resolve the conflict" — reasoning that conflict resolution belongs downstream, to "the coordinator or the consumer." That's backwards for this question: punting an unresolved contradiction into the final synthesized answer is itself the reliability failure the exam is testing for.

## Strengths

code_exploration (88%) held up despite the fork_session and replace_all misses — everything else in that domain (context-efficient search strategies, incremental auth-flow mapping, scratchpad use for long sessions, sub-agent spawning with summarized context, session resumption after external file changes) was answered correctly, including several multi-option "big questions" the transcript shows genuine deliberation on. research_pipeline (86%) and extraction_pipeline (87%) were both solid too — parallel subagent dispatch, claim-source structured mapping, schema-constrained fields, calculated-vs-stated-total integrity checks, and confidence-threshold validation-by-segment were all correct, showing the underlying extraction/orchestration concepts are genuinely internalized; the misses in those two domains were narrow (2 each) rather than systemic.

## Course-topic performance

Judged by actual tested concept against existing chapter notes, not the exam's own domain labels. This attempt's source has no question numbers, so this table covers only the 10 missed questions plus the domains they sit in — not a full per-question sweep of all 60 (unlike prior attempts where numbered questions made that tractable).

| Chapter note | Missed questions mapped |
|---|---|
| [text-chunking-strategies.md](../../ccaf-learning/claude-api/rag-and-agentic-search/text-chunking-strategies.md) | Long-contract chunk-vs-summarize miss |
| [agents-and-workflows.md](../../ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md) | Query-routing miss (dynamic coordinator judgment is covered there, though not the exact fast-path framing) |

## Gap-topic performance

No missed questions this attempt mapped cleanly onto an existing gap-topic note under `ccaf-learning/` (checked `structured-data-extraction/*`, `tool-choice-forcing/`, `worktree-coordination/`) — the extraction-format-variance miss is adjacent to `structured-data-extraction/confidence-calibration.md`'s mention of few-shot as a prompt-level accuracy fix, but that note doesn't cover the varied-source-format scenario directly, so it's not counted as a match.

**Coverage gaps surfaced this attempt** (real CCAF scope, no existing note): `fork_session` / session branching mechanics (Claude Code Agent SDK), MCP tool result error signaling (`isError`, structured retryable metadata vs. plain-text/success-disguised errors), and Claude Code's Edit tool `replace_all` vs. Read+Write tradeoffs specifically for insertion tasks. Worth a research pass if you want dedicated notes — none were force-fit here.

## Survival guide updates (draft — push back if these don't hold up)

Updated [`exam-prep-survival-guide.md`](../study-guide-weak-spots/notes/exam-prep-survival-guide.md):

- **"Missing capability vs. already-adequate piece"** — added the query-routing and escalation-trigger-design misses as new worked examples under "Existing piece needing a fix, not a new layer." Both are hardcoded-rule-over-model-judgment misses, same shape as try 4 Q30.
- **New shorter note**: "MCP tool errors belong in the tool result's error channel, not a disguised success or external control logic" — drafted from the `lookup_order` and `process_refund` misses, both from this same attempt. This is a first appearance, not a cross-attempt pattern yet — flagged as a draft worth watching for recurrence before treating as settled.

## Follow-ups from the meeting's Next steps (owner: claude / the group / GL)

The transcript's own "Next steps" section flagged four items during the sitting; resolved here since they're tied directly to this attempt's questions, not separate research tasks.

- **`(owner: claude)` Why do temporary DB failures return as a "successful" response? (Q11, `lookup_order`)** — same root cause as the `isError` pattern above: the tool wraps both a permanent failure ("order not found") and a transient one ("temporary DB timeout") in one uniform success-with-status-field shape, which erases the retryable/non-retryable distinction the calling agent needs to decide retry-vs-escalate. The fix (and the correct answer, B) is the tool result's own error channel — `isError: true` plus a category field — not a status field bolted onto a success payload.
- **`(owner: claude)` Edit tool parameters, second brain or Claude docs** — confirmed via [Claude Code Docs — Tools reference](https://code.claude.com/docs/en/tools-reference): `Edit` takes `file_path`, `old_string`, `new_string`, and optional `replace_all` (boolean, default `false`). Exact string match only, no regex/fuzzy matching; `old_string` must be unique in the file (or given enough surrounding context to disambiguate) unless `replace_all: true`, which replaces *every* occurrence. `replace_all` is designed for file-wide rename-style edits, not targeted single-location insertion — confirming Q32's correct answer (Read + insert + Write) over forcing `replace_all` on a merely-repetitive (not actually uniform-intent) pattern. No existing `ccaf-learning/` note covers this — a gap-topic note is optional if you want it on record.
- **`(owner: the group)` Investigate Question 51** — the "resume the subagent from its previous transcript, inform it about renamed functions" question (code_exploration), which Bruno second-guessed live in the transcript ("I'll be shocked if it's not D... if it's not D, I got that wrong"). Checked against the results: **answered correctly** (D), no `Correct:` override shown. The in-the-moment doubt was unfounded.
- **`(owner: GL)` Did the error rate increase from Q46–60 vs. earlier?** — mapped all 10 misses to their position in the 60-question sequence: 1, 11, 14, 15, 17, 22, 32, 39, 46, 49. Q1–45: 8/45 wrong (17.8%). Q46–60: 2/15 wrong (13.3%). **Error rate did not increase in the back third — it dropped.** The "patience decreasing" feeling reported live in the transcript didn't translate into more mistakes this attempt, unlike try 5's back-third pacing pattern.

## Answer-by-answer coaching

Every question in exam order. The source PDF has no question numbers — the numbering below is this note's own, matching the sequence used in the "error rate" follow-up above. Wrong answers lead with the correct option quoted; correct answers get one sentence.

**Q1 — code_exploration — WRONG.** Exploring two refactoring approaches (microservice extraction vs. in-place) in depth from yesterday's analysis. Correct: **"Use `fork_session` to create two branches from yesterday's analysis, exploring one approach in each fork."** Two fresh sessions with a manually-typed summary (your pick) throws away everything the agent actually discovered — file locations, reasoning trail, edge cases noticed — and replaces it with your compressed recollection of it. `fork_session` keeps the full original context intact in both branches, which is exactly what "propose specific code changes for each" needs. You answered a near-identical question later (Q43) correctly with `fork_session` — the concept is there, it just didn't fire here.

**Q2 — code_exploration — correct.** Grep for auth entry points then follow imports/calls incrementally is the context-efficient way to map an 800+ file codebase — read only what the trail actually leads to, not the whole tree.

**Q3 — code_exploration — correct.** Summarizing rendering findings and spawning a fresh sub-agent for physics exploration keeps the two investigations from bleeding together — the stale "typical rendering patterns" phrasing was the tell that the original context was already overloaded.

**Q4 — code_exploration — correct.** Searching for the exact error string first means you only open files that are actually part of the error's path, not the whole service.

**Q5 — customer_support — correct.** Flagging possibly-stale data and verifying/escalating instead of asserting it as fact avoids confidently repeating information that might already be wrong.

**Q6 — code_exploration — correct.** Sparse tool descriptions ("extracts a function from code") give the model no reason to prefer the MCP tool over Bash/Write — enriching the description with when/why/expected I/O is the fix, not more configuration.

**Q7 — code_exploration — correct.** Reading entry points and structure before searching gives the exploration a map to search *within*, instead of grepping blind across an unfamiliar repo.

**Q8 — code_exploration — correct.** Searching within the file and reading only the target region + dependencies avoids loading a multi-thousand-line file into context for one function.

**Q9 — customer_support — correct.** A fresh session with an injected structured summary plus new tool calls avoids both the stale-data trap (old session) and total context loss (no summary) for a returning customer.

**Q10 — customer_support — correct.** Acknowledging frustration, stating policy plainly, and offering what does exist resolves the interaction honestly without pretending a policy exception is possible.

**Q11 — customer_support — WRONG.** `lookup_order` backend errors ("order not found," temporary DB failure). Correct: **"Return the error message in the tool result content with the `isError` flag set to `true`."** A success response with a status field (your pick) tells the calling agent "this worked" when it didn't — there's no reliable way for the agent to distinguish a real order state from a failure state buried in a status string. `isError: true` uses the tool-result protocol's actual purpose-built error channel, so the agent can reason about the failure directly instead of parsing ad hoc status text. You flagged this yourself live in the transcript ("temporary database failures returned as a successful response... it's kind of weird") — the instinct that something was off was right; the answer picked wasn't.

**Q12 — customer_support — correct.** A structured handoff (customer details, order info, identified issue) before calling `escalate_to_human` gives the next handler everything needed without re-investigating from scratch.

**Q13 — research_pipeline — correct.** An explicit budget plus a coverage check gives the research loop a concrete stop condition instead of letting it spawn follow-ups indefinitely.

**Q14 — research_pipeline — WRONG.** Query routing for a "diverse and evolving" query distribution. Correct: **"Have the coordinator analyze each query and dynamically decide which subagents to invoke based on its assessment of query requirements."** A hardcoded fast-path (your pick — bypass subagents for factual questions, full pipeline for everything else) is a fixed two-bucket rule; it can't adapt as the query distribution keeps changing shape, which the question explicitly says it does. The coordinator already exists and is capable of judgment — dynamic per-query analysis handles both the simple and complex cases (and everything in between) without needing to predict every category in advance. You considered this option directly in the transcript and talked yourself out of it by reading "complex research benefits from the full pipeline" as evidence *for* the rigid split, when it's really just one of the shapes the coordinator's judgment needs to cover.

**Q15 — extraction_pipeline — WRONG.** A contract too long for one context window, fields needed from across the whole document. Correct: **"Chunk the document with slight overlap, extract per chunk, then merge and reconcile the fields."** Summarizing first (your pick) is lossy by construction — a summarization pass can paraphrase away or drop exactly the specific field values extraction later needs, before extraction ever gets a chance to run on the original text. Chunking with overlap keeps the model working from the source language directly in every chunk; the overlap prevents fields from being cut exactly at a chunk boundary.

**Q16 — customer_support — correct.** Escalating with full history and what's been tried after three failed attempts avoids making the customer re-explain everything to a human agent.

**Q17 — extraction_pipeline — WRONG.** Valid JSON but empty citation/methodology fields, because source documents present that info in varied formats. Correct: **"Add few-shot examples demonstrating extractions from documents with varied structures — showing how to identify citations in different formats and locate methodology details across section types."** A regex-based post-processing layer (your pick) is new infrastructure that has to independently re-solve the same format-detection problem the model is already positioned to solve directly — and it needs to be maintained/extended every time a new citation format shows up. Few-shot examples fix the actual gap (the model hasn't seen enough format variety in-context) at the source, inside the extraction call itself.

**Q18 — research_pipeline — correct.** Each agent outputting structured data that separates content summaries from source metadata (URLs, doc names, page numbers) keeps attribution intact through the summarization step instead of losing it.

**Q19 — research_pipeline — correct.** The synthesis agent failing with "no research findings provided" points straight at the coordinator's prompt-construction step, not the synthesis agent's own logic — it can only work with what it's actually given.

**Q20 — research_pipeline — correct.** Merging and de-duplicating overlapping findings before synthesis, keeping one cited instance each, avoids the final report repeating the same fact multiple times with different citations.

**Q21 — extraction_pipeline — correct.** Forcing `tool_choice` to `extract_metadata` guarantees the DOI-producing call happens before the DOI-dependent enrichment tools are even offered as options.

**Q22 — customer_support — WRONG.** Frustrated customer demanding a human immediately; agent hasn't called any tools yet. Correct: **"Acknowledge the frustration and ask one targeted question to understand the specific issue before escalating."** Calling `get_customer`/`lookup_order` first (your pick) is a reasonable instinct but skips past the customer entirely at the exact moment they said they feel unheard — gathering account data doesn't address "I want to talk to a real person NOW," it just delays a response to it. A short acknowledgment plus one targeted question costs almost no time and directly engages with the stated frustration before any tool call or escalation. Your transcript shows this was a genuine A-vs-D toss-up in the moment — the tool-investigation instinct won, but "has anyone actually acknowledged what I said" is the thing an upset customer is checking for first.

**Q23 — research_pipeline — correct.** Having the coordinator handle straightforward summarization directly (it already has the findings in context) instead of re-spawning a synthesis subagent every time avoids re-transmitting 80K+ tokens for something the coordinator can already answer.

**Q24 — extraction_pipeline — correct.** Verifying line items sum to the extracted total, flagging/retrying on mismatch, catches both OCR errors and model extraction mistakes before they reach downstream accounting systems.

**Q25 — code_exploration — correct.** Reading the library and wrapper modules first to find every renamed alias, then grepping for each name, is the only way to catch callers that go through a renamed wrapper instead of the original function name.

**Q26 — research_pipeline — correct.** Splitting the generic `analyze_document` tool into purpose-specific tools (`extract_data_points`, `summarize_content`, `verify_claim_against_source`) gives each one a defined input/output contract, instead of leaving the model to guess what a free-text instruction should return.

**Q27 — code_exploration — correct.** A scratchpad file recording key findings gives long exploration sessions a durable reference point instead of relying on conversation memory that degrades over 30+ minutes.

**Q28 — customer_support — correct.** Summarizing earlier turns into a narrative while preserving full history only for the active issue keeps all three raised issues addressable without blowing the context budget on all of them at full fidelity.

**Q29 — code_exploration — correct.** Confirming in the actual current code where the auth check runs (not trusting the README) is the only way to be sure before changing behavior that documentation might describe inaccurately or out of date.

**Q30 — research_pipeline — correct.** Specifying research goals and quality criteria instead of exact procedural steps lets the subagent adapt its search strategy when a pre-specified approach fails or an emerging topic doesn't match expected patterns.

**Q31 — customer_support — correct.** Acknowledging frustration and offering to complete the already-confirmed-eligible return immediately (or escalate) resolves the actual blocker instead of making the customer wait through another round of "let me check."

**Q32 — code_exploration — WRONG.** Inserting a new function into a 150-line file with repetitive docstrings/patterns, where `old_string` can't find unique text. Correct: **"Use Read to load the file, add the function at the appropriate location, then Write the updated file."** `replace_all` (your pick) is designed for renaming a symbol consistently everywhere it appears — forcing it to match a "common pattern" for a single precise insertion risks corrupting every other occurrence of that pattern in the file, not just the one you want. Read+Write gives you full, explicit control over exactly where the new function lands, with no risk of an over-broad match touching unrelated code. Confirmed against the Claude Code docs: `Edit`'s `old_string` must be unique (or given more surrounding context) unless `replace_all: true` replaces *every* match — it's not a targeted-insertion tool.

**Q33 — customer_support — correct.** A structured summary (customer ID, root cause, refund amount, recommended action) gives a human agent who has no transcript access everything needed to resolve the case without re-investigating.

**Q34 — research_pipeline — correct.** Rendering each content type appropriately (financial data as tables, news as prose) instead of flattening everything to bullet points preserves the information each source type actually needs to communicate clearly.

**Q35 — extraction_pipeline — correct.** Retries are least effective when the missing data (the full co-author list) simply doesn't exist anywhere in the input — no amount of re-prompting the model on the same input can produce information that was never there.

**Q36 — customer_support — correct.** Structured error metadata (`errorCategory`, `isRetryable`, a cause description) lets the agent actually distinguish transient-worth-retrying from permanent-not-worth-retrying from needs-clarification, instead of guessing from a uniform "Operation failed" message.

**Q37 — research_pipeline — correct.** Requiring subagents to output structured claim-source mappings that the synthesis agent preserves and merges keeps attribution intact through the combination step, the same underlying fix as Q18 applied to a slightly different failure mode.

**Q38 — extraction_pipeline — correct.** Redesigning the schema so amended fields capture multiple values (each with source location and effective date) lets the extraction represent "this term changed" instead of forcing a single silently-chosen value.

**Q39 — customer_support — WRONG.** Designing the trigger for `escalate_to_human`. Correct: **"Instruct the agent to escalate when the customer requests a human, when the issue requires policy exceptions, or when the agent cannot make meaningful progress."** A three-failed-calls threshold (your pick) is a count-based proxy that misses the two most common real escalation reasons entirely — an explicit request for a human, or a policy limit the agent has no authority to override — and it can both over-trigger (three failures on something ultimately easy) and under-trigger (zero failures, but the customer already asked for a person). Criteria-based rules track *why* escalation is actually needed; a call-count doesn't.

**Q40 — extraction_pipeline — correct.** Requiring an ISO date and flagging ambiguous inputs for review (instead of guessing March vs. April) avoids a silent, unverifiable wrong answer for genuinely ambiguous dates.

**Q41 — extraction_pipeline — correct.** Constraining the field to the five allowed enum values and rejecting anything else stops the model from inventing new priority labels outside the defined set.

**Q42 — customer_support — correct.** Answering a directly-knowable question immediately, offering escalation only if more is needed, avoids adding unnecessary friction for something the agent can already resolve.

**Q43 — code_exploration — correct.** Resuming with `fork_session` to create a separate branch per testing strategy lets both approaches be developed independently from the same 23-file analysis without re-doing the exploration — the same mechanic Q1 missed, applied correctly here.

**Q44 — code_exploration — correct.** Dynamically generating investigation subtasks as the agent discovers more about the error path (routing → middleware → business logic → database) adapts the plan instead of committing to a fixed decomposition before anything is known about where the bug actually lives.

**Q45 — extraction_pipeline — correct.** Checking accuracy by document type and field (not just in aggregate) before automating high-confidence extractions catches segments where the 97% aggregate accuracy might be hiding a much weaker sub-population.

**Q46 — research_pipeline — WRONG.** Two subagents return conflicting figures for the same metric, each with moderate confidence. Correct: **"Run a focused check that re-fetches the metric from the primary source and resolves the conflict before synthesizing."** Including both numbers and letting the reader decide (your pick) pushes an unresolved contradiction into the final answer instead of resolving it — that's the reliability failure the question is testing for, not an acceptable transparency move. A moderate-confidence conflict on a single metric is cheap to verify directly against the primary source; punting it downstream just relocates the uncertainty instead of removing it. Your transcript shows you rejected the verification option outright ("I don't think that's going to work... conflict resolution belongs to the coordinator or the consumer") — reasonable-sounding, but backwards here: the coordinator *is* the one with the ability to verify before the reader ever sees the number.

**Q47 — extraction_pipeline — correct.** A strict output schema plus explicit format-normalization rules in the prompt handles inconsistent menu formatting (`$12` vs `12.00`, icons vs text) at the point of extraction instead of downstream.

**Q48 — code_exploration — correct.** `--resume auth-deep-dive` loads the exact named session directly, avoiding any ambiguity from having worked on three other codebases since.

**Q49 — customer_support — WRONG.** `process_refund` errors: transient technical errors (5%) vs. permanent business errors (12%), currently both plain text. Correct: **"Return structured error responses with `retryable: false` for business errors and a customer-friendly explanation for Claude to use."** Tool-level auto-retry for technical errors only (your pick) still keeps the retry decision as external logic sitting beside the tool call, which has to be kept in sync by hand as error types change. Putting `retryable` as structured metadata directly on the error response lets the calling agent make the retry-vs-not decision itself from the tool result, the same underlying fix as Q11 applied to a second tool.

**Q50 — customer_support — correct.** A hook that intercepts tool calls and blocks any refund over $500 before it executes is the only way to get *guaranteed* compliance — system-prompt instructions are guidance the model can still deviate from, which is exactly what the 3% failure rate showed.

**Q51 — code_exploration — correct.** Resuming the subagent from its previous transcript and explicitly informing it which two functions were renamed preserves the 30 minutes of prior analysis while patching the one thing that's now stale.

**Q52 — extraction_pipeline — correct.** A `calculated_total` field (model-summed line items) alongside the extracted `stated_total`, flagged on mismatch, catches both OCR errors and extraction mistakes the same way Q24's line-item check does, applied to invoices specifically.

**Q53 — research_pipeline — correct.** Spawning parallel document-analysis subagents per precedent, then aggregating, cuts the 3-minute sequential analysis down while the coordinator still gets discrete per-precedent results it can monitor and debug.

**Q54 — extraction_pipeline — correct.** Adding an "other" enum value with a separate `property_type_detail` string field handles new property types as they keep appearing, without either rejecting valid-but-uncommon listings or endlessly growing the enum.

**Q55 — extraction_pipeline — correct.** Submitting batches every 4 hours keeps well within the 30-hour SLA even accounting for the Message Batches API's up-to-24-hour processing window, with margin for the API's own delay.

**Q56 — research_pipeline — correct.** Dispatching eight independent subagents in parallel, each returning a short structured summary with citations, gets full coverage of non-dependent sources fast without dumping eight raw research dumps into the coordinator's context.

**Q57 — research_pipeline — correct.** Requiring subagents to include publication/data-collection dates in their structured outputs lets the synthesis agent recognize "35% in 2024 vs. 18% in 2022" as growth over time instead of a contradiction.

**Q58 — extraction_pipeline — correct.** Few-shot examples with standardized material-description formats reduce inconsistent extraction ("cotton blend" vs. "Cotton/Polyester mix" vs. omitted) by showing the model the exact output shape wanted.

**Q59 — extraction_pipeline — correct.** Field-level confidence scores calibrated against a labeled validation set let reviewer attention go where it's actually needed, instead of spreading fixed 20% capacity evenly across extractions regardless of risk.

**Q60 — code_exploration — correct.** Resuming the session while explicitly telling the agent which 3 of the 12 previously-read files changed overnight targets re-analysis at exactly what's stale, without re-doing the whole hour of prior work.

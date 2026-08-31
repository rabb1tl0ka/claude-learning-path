# CCAF exam survival guide: cross-cutting heuristics

A growing collection of exam-taking heuristics that cut across domains and topics — the kind of thing that comes out of a post-mock-exam discussion, not a single chapter note. Distinct from the topic-specific notes under `ccaf-learning/` (those teach a concept; this file teaches *how to reason under multiple-choice pressure* when several of these concepts could plausibly apply). Each heuristic below links out to a deeper single-topic note when one exists.

## Talking yourself out of a correct first instinct

Source: follow-up discussion after the try 5 mock exam (2026-08-23). This is the single most-evidenced pattern across all five attempts — try 1's own analysis (`../../attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md`) independently named it a week before try 5 and it kept recurring anyway, which is exactly why it's worth a dedicated entry instead of living as a footnote inside another heuristic.

### The trap

The correct answer gets identified — out loud, or as a first instinct — and then gets talked back out of, replaced by something else. This isn't a knowledge gap (the right fact was already surfaced); it's a mid-reasoning override. Evidence across every attempt so far:

- **Try 1** Q14, Q17, Q33 — said the correct answer, then switched. Q17 is the clearest: *"I would replace the pro[se] severity description with concrete code examples"* (correct), then got pulled into a polyglot-codebase tangent and picked "second model pass" instead.
- **Try 1** Q16 — *"I love this, I love this"* right before switching from the correct answer (stratified sampling) to a fancier document-format classifier.
- **Try 3** Q59 — *"it could be C"* (correct), overridden by an overgeneralized rule: *"a coordinator should never handle work, actual work."*
- **Try 3** Q17 (revisit) — audibly circling the correct answer, trails off into "I don't think so, I'm going to go with this," submits the wrong one under time pressure.
- **Try 4** Q30 — *"I'm afraid that option B might imply that the tool is not being used"* — rejected the correct answer over a misreading of what it implied.
- **Try 5** Q36 — considered the correct PostToolUse hook, downgraded it in his own paraphrase ("this will only validate, wouldn't do it") before picking a rules file instead.
- **Try 5** Q57 — explicitly rejected the "shared context" framing ("we don't need sh[ared] context here") when shared context was the point.
- **Try 6** Q02 (transcript-confirmed) — reasoned live to the correct answer ("keep tool_use, set tool_choice to force the specific extraction tool by name"), then doubted it ("doesn't necessarily guarantee structured output... depends on the tool") and reverted to the first-read-wrong option: *"I'm going to keep B, it's my first instinct anyway."* Note the irony — "first instinct" was invoked to justify keeping the option that was *not* the correct first instinct; the actual correct reasoning had already been spoken and set aside.

### The four pulls, and which one deserves to win

Not every override has the same cause — four distinct pulls have shown up so far, and only one of them should ever actually move you off a first instinct:

1. **Semantic imprecision** — a close-but-different word gets read as something it doesn't say. "Validate" quietly becomes "wouldn't do it" (try 5 Q36); "model-driven decisions" gets read as "stop using the tool" (try 4 Q30). This is the one legitimate reason to reconsider — *if* re-reading the option's literal text confirms the imprecision. Most of the time it doesn't; the gloss was just wrong.
2. **Rule over-generalization** — a correct conditional rule ("coordinators shouldn't do specialist work") gets applied as an absolute and overrides a correct specific instinct that actually fit the rule's real (narrower) scope (try 3 Q59). Almost never a legitimate reason to switch — the fix is to recall the rule's actual precondition, not to apply it more broadly.
3. **Aesthetic pull toward the more sophisticated-sounding option** — *"I love this"* (try 1 Q16) is a tell, not a signal. Never a legitimate reason to switch on this exam — complexity reads as rigor, but the exam rewards the opposite.
4. **Anecdotal override** — a specific remembered instance of past tool behavior gets trusted over the option's literal, stated mechanism. Try 6 Q03 (transcript-confirmed): correctly read "expand the Edit search string for more context" as the right recovery path, then overrode it with *"I actually think that's how Claude Code works... I've seen it"* and picked Bash + `sed` instead — a real memory, but not evidence about what *this question's* options actually say. Never a legitimate reason to switch — a remembered instance generalizes worse than the option text in front of you.

### The counter-strategy

Run these, in order, before letting a second thought override a first instinct:

1. **Restate the option's mechanism literally**, in the option's own words, not your compressed gut-gloss on it. If the restatement needs a qualifier that isn't actually in the text ("only validates," "wouldn't do it," "might imply"), that qualifier is the thing to distrust, not the option.
2. **Run the disambiguation checklist** from [Missing capability vs. already-adequate piece](#missing-capability-vs-already-adequate-piece--dont-add-new-machinery-when-something-already-does-that-job) below — it's the concrete anchor that replaces a vague "does this feel right" with an actual yes/no test, which is exactly the clarity that's missing in the moment an override happens.
3. **Treat "this sounds more advanced/thorough" as a warning**, not a point in favor.

**The behavioral rule:** the burden of proof is on whatever is pulling you away from the first read, not on the first read itself. The first instinct has been right more often than the override, across all six attempts.

### When reciting the checklist isn't enough on its own

Try 6 Q11 (transcript-confirmed) is a harder case than the four pulls above: the correct answer was named out loud ("I would pick [Grep] to find which of the three occurrences is the correct one"), *and* the counter-strategy itself was recited correctly ("state the option's mechanism literally, not your gut-gloss") — and the submitted answer was still the wrong one (`sed`). Knowing the rule and stating it isn't the same as committing to it under time pressure; the failure moved from *reasoning* to *execution*. There's no clean fix for this yet beyond naming it, but the practical takeaway: once you've said the correct answer's letter out loud, select that letter immediately — don't let the click happen several sentences of hedging later.

## Missing capability vs. already-adequate piece — don't add new machinery when something already does that job

Source: follow-up discussion after the try 5 mock exam (2026-08-23), triggered by the Q36/Q40/Q58 misses and the "reaching for more machinery instead of the intended structural/judgment fix" failure pattern that recurs across try 2 (`../../attempts/claudecertificationguide-mockexam-try2-2026-08-18-analysis.md`), try 4 (`../../attempts/claudecertificationguide-mockexam-try4-2026-08-22-analysis.md`), and try 5 (`../../attempts/claudecertificationguide-mockexam-try5-2026-08-23-analysis.md`).

### The trap

"Reach for a structural fix" (hooks, validation checkpoints, Plan Mode) and "reach for a fancier architecture" (a new coordinator, a routing layer, a new classifier model, splitting into more agents) both *feel* like the same instinct — "add more machinery" — but the exam rewards one and punishes the other. Conflating them is what causes flip-flopping between picking the boring fix and the impressive-sounding one.

They are actually two different axes:

- **Missing capability → add it.** When nothing in the described system today enforces a guarantee — no hook exists, no validation step exists, no planning discipline was used before touching 12 cross-module dependencies — the fix is to add one of the small number of standard mechanisms the course actually teaches (a PreToolUse/PostToolUse hook, Plan Mode, a schema constraint, a validation checkpoint). This is where reaching for structure is *correct*.
- **Existing piece is under-specified → fix that piece, don't wrap a new layer around it.** When something already exists and is basically doing its job but is misconfigured — a tool description that's too sparse, a two-agent split that used to work fine before scope crept in, a decision tree that's too rigid — the fix is to correct *that specific piece*, not introduce a new coordination layer, a new classifier, a new agent, or a new architectural tier on top of it. This is where reaching for structure becomes over-engineering.

### The test

Before picking an answer, ask: **does this system already have a piece whose job this is?**

- **Yes** → fix that piece directly (rewrite its description, restore its original scope, recalibrate its threshold). Don't wrap a new layer around it.
- **No** → then, and only then, add one of the standard mechanisms this specific gap requires (hook, Plan Mode, schema constraint, validation step) — not a bespoke new architecture invented for the occasion.

"Fancier" is the tell for over-engineering specifically when it means *new infrastructure, agents, models, or routing layers*. It's the right call only when it means *the specific standard mechanism this gap requires, and genuinely nothing plays that role yet*.

**Important exception, confirmed by try 6 Q59 (transcript):** splitting an existing overloaded tool/agent into narrower ones is *not* "new machinery" in this heuristic's sense, even though it sounds like "adding more pieces." It's redrawing the boundary of a piece that already exists — see [Tool/agent-granularity direction](#toolagent-granularity-direction--which-way-should-the-count-actually-move) below, which is the more specific rule for this exact shape of question. Try 6 Q59 explicitly invoked this heuristic to reject the correct answer ("this would be overengineering... this is that machinery all over again") — a real misapplication: the two heuristics can look like they contradict each other in the moment, but they don't, because they answer different questions. This one asks "does something already do this job" (yes → fix it in place, no → add the missing mechanism); granularity asks "is the *existing* piece serving too many unrelated jobs at once" (yes → split it — that's fixing the existing piece, not adding a new layer on top of it).

### Worked examples

**Existing piece needing a fix, not a new layer:**
- Try 1 Q1 — a routing-classifier instinct picked over restoring a role-scoped subagent split that already existed and worked.
- Try 2 Q57 — an MCP tool underused, agent falls back to Bash. Correct fix: enhance the tool's own sparse description. Wrong instinct: a system-prompt instruction telling the agent to prefer the MCP tool — the same trap, a full exam attempt (five days) before try 5 hit the near-identical Q40.
- Try 2 Q58 — `.claude/rules/` file banning mocks outright, when the actual gap was underspecified CLAUDE.md criteria (which kinds of tests need real DB connections). Over-broad enforcement bolted on instead of a precise standard.
- Try 4 Q26 — a text classifier bloated with image-analysis responsibilities. Correct fix: restore the original two-agent split (text classifier back to text-only). Wrong instinct: split into *three* narrower specialists — solving an over-scoped-agent problem by adding more agents.
- Try 4 Q30 — a hardcoded moderation decision tree missing satire and sophisticated spam. Correct fix: replace the hardcoded tree with model-driven judgment. Wrong instinct: add more granular category labels — a hardcoded tree can't adapt to nuance no matter how fine the labels are.
- Try 7 (research_pipeline) — a coordinator dispatching to a fixed set of subagents, with query complexity "diverse and evolving." Correct fix: have the coordinator itself analyze each query and dynamically decide which subagents to invoke. Wrong instinct: a hardcoded fast-path (bypass subagents for factual questions, full pipeline for everything else) — a two-bucket rule can't track a query distribution that keeps changing shape, which is exactly what the coordinator's own judgment is for.
- Try 7 (customer_support) — designing the escalation trigger for `escalate_to_human`. Correct fix: escalate on stated criteria (customer requests a human, issue needs a policy exception, agent isn't making progress). Wrong instinct: a count-based threshold (escalate after three consecutive failed tool calls) — misses the two most common real triggers entirely (explicit request, policy limit) and can both over- and under-fire relative to them.
- Try 5 Q40 — an agent underusing a sparsely-described CRM MCP tool, falling back to Grep. Correct fix: expand the tool's own description. Wrong instinct: add a system-prompt instruction ("always use CRM, never Grep") — a hardcoded exclusion rule that breaks if the tool is renamed or a legitimately-Grep-worthy task shows up later.
- Try 9 (research_pipeline) — follow-up summarization queries ("summarize what we learned about market trends") each re-spawn the synthesis subagent, re-transferring 80K+ tokens the coordinator already has in its own context. Correct fix: have the coordinator answer directly using context it already holds, reserving subagent spawning for genuinely new/complex analysis. Wrong instinct: enable prompt caching on the subagent — a real technique, but it optimizes the cost of doing unnecessary work instead of asking whether the extra hop (a piece that already exists — the coordinator itself — could just do the job) was needed at all.
- Try 8 (customer_support) — a customer's three issues (refund, subscription, payment update) risk falling out of context as the session nears its limit. Correct fix: summarize earlier turns into a narrative, keeping full message history only for the currently-active issue — the conversation itself already tracks all three, it just needs smarter compaction. Wrong instinct: extract and persist structured issue data (order IDs, amounts, statuses) into a separate context layer — a new piece of infrastructure bolted on to do a job ordinary context summarization already does.
- Try 8 (research_pipeline) — a synthesis agent inconsistently handles conflicting subagent uncertainty (one hedges with a methodology caveat, one gives a confidence interval), sometimes flattening it into false confidence, sometimes over-hedging into vagueness. Correct fix: instruct the synthesis agent itself to structure the report with explicit sections distinguishing well-established from contested findings, preserving each source's own characterization. Wrong instinct: build a confidence-calibration layer that normalizes every subagent's uncertainty language into a standardized 0.0-1.0 score and weight-averages them — manufacturing a false sense of precision (no ground-truth validation set backs that normalization, see [`confidence-calibration.md`](../../ccaf-learning/structured-data-extraction/confidence-calibration.md)) and losing exactly the methodological nuance ("varies," "±7B, 95% CI") the question says is getting lost today.

**Missing capability that genuinely needed adding:**
- Try 5 Q58 — a 3-step pipeline where step 2 occasionally misclassifies document type, silently corrupting step 3. Correct fix: add a validation checkpoint between steps 2 and 3. There was no mechanism catching this today; more few-shot examples (the tempting fix) lowers the error rate but can never guarantee zero, so the checkpoint is a genuinely new, necessary piece — see [`prose-clarity-vs-examples-fix.md`](prose-clarity-vs-examples-fix.md) for the sibling case of when examples *are* the right fix vs when they're a band-aid over a missing structural piece.
- Try 5 Q11 / Q43 — irreversible actions (account deletion, international transfers) with no enforcement gate today. Correct fix: a PreToolUse hook that blocks execution until approval/AML verification. Nothing else in the system plays this role.
- Try 4 Q32 / try 5 Q17 — a multi-dependency migration attempted via direct execution with no upfront mapping. Correct fix: Plan Mode. No planning discipline existed yet, so this isn't adding an extra layer on top of something adequate — it's adding the step that was skipped.
- Try 9 (code_exploration) — an agent investigating untested code paths across 45 files starts losing accuracy after reading only 8 of them (forgetting earlier patterns, still hasn't located test files or traced flows). Correct fix: spawn subagents for specific sub-questions ("find all test files," "trace refund flow dependencies") while the main agent coordinates and preserves high-level understanding. Wrong instinct: switch to Grep instead of reading full files — a tune-up of the *existing* single-agent approach, when the actual gap is that nothing yet decomposes the investigation across separate context windows. Same shape as try 5 Q58's validation checkpoint: a "make the current approach leaner" fix can't solve a problem that's structurally about one agent's context being asked to hold too much at once.

**The edge case: building from scratch.** When a question describes designing a brand-new system with no existing structure to preserve (e.g. an architect building a support system from zero), "does this system already have a piece for that job" is moot — there's nothing yet, so proposing an architecture (hub-and-spoke, scoped specialist agents via a coordinator) is the intended answer, not over-engineering. The over-engineering trap specifically applies when an existing, working design is described and the complaint is about a symptom (accuracy dropped, slow, wrong tool picked) — that's the signal the fix is a tune-up, not a rebuild.

## Tool/agent-granularity direction — which way should the count actually move?

Source: cross-attempt review (2026-08-23) of try 1 (`../../attempts/claudecertificationguide-mockexam-try1-2026-08-16-analysis.md`), try 2 (`../../attempts/claudecertificationguide-mockexam-try2-2026-08-18-analysis.md`), and try 3 (`../../attempts/claudecertificationguide-mockexam-try3-2026-08-22-analysis.md`) — the same underlying miss shows up in both directions, which is exactly why it keeps tripping: there's no single "add more" or "add less" default, only a rule for which way a given scenario points.

### The rule

- **Split** when one tool/agent is serving 3+ genuinely unrelated purposes (web scraping + document parsing + code analysis; a text classifier that grew image-analysis responsibilities). The tell: the description or prompt is long and covers unrelated jobs, and picking the wrong one of its jobs is the failure mode.
- **Consolidate** when several near-duplicate tools/agents overlap in purpose and the model can't tell them apart. The tell: multiple options exist for essentially the same job, and picking the wrong *duplicate* is the failure mode.
- **A structural detail that doesn't help either way:** splitting tools across separate MCP servers doesn't reduce the agent's actual choice set — all connected tools appear in one flat list to the agent regardless of which server they came from. Server boundaries are invisible to the model; only actually removing/merging tools changes what it has to choose between (try 2 Q24).

### Worked examples

- Try 1 Q1 — see above (missing-capability heuristic) — restoring an existing role-scoped split, not adding a router.
- Try 2 Q11 — a tool doing 3 distinct jobs needed splitting into purpose-specific tools; picked "improve the description" instead, patching the symptom rather than the actual one-tool-three-jobs cause.
- Try 2 Q24 — 22 near-duplicate tools needed consolidating to ~4; picked splitting them across two MCP servers, which doesn't touch the agent's real choice set at all.
- Try 3 Q52 — a select-3 question on fixing tool misrouting; "merge into one generic tool" was picked as one of the three fixes, but the guide's own fix moves the *opposite* direction (splitting a generic tool into purpose-specific ones) — direction-confusion in a single question.
- Try 6 Q59 — a single `analyze_content` tool doing web scraping, document parsing, and code analysis indiscriminately; picked "improve the description" instead of splitting into purpose-specific tools (`extract_web_results`, `parse_document`, `analyze_code`). Near-identical to try 2 Q11 — same trap, same wrong instinct, five attempts apart.

See also [`prose-clarity-vs-examples-fix.md`](prose-clarity-vs-examples-fix.md#cases-where-concrete-examples-are-the-right-fix)'s "tool granularity/purpose confusion" case for the instruction-level version of the same idea.

## Hook timing depends on data availability, not pipeline position

Source: try 3 mock exam (2026-08-22), Q20 and Q32 (`../../attempts/claudecertificationguide-mockexam-try3-2026-08-22-analysis.md`).

### The trap

"Earlier in the pipeline" feels like it should mean "more root-cause, more robust, prevents the problem at its source." For hook timing specifically, that intuition is backwards. PreToolUse fires *before* the tool runs — there's no output yet to check. PostToolUse fires *after* — the completed result exists and can be validated or acted on. Which one is correct depends entirely on what data the hook actually needs, not on which one sounds more preventative.

### The test

Ask: **does the hook need to see the result of the action, or only the request going in?**

- Needs to see the *result* (file content just written, a normalized field, whether a rollback section is present) → PostToolUse. Nothing to check yet if you fire before the write happens.
- Needs to *block or modify the request itself* before it executes (an irreversible action, a parameter that must be validated before the call is made) → PreToolUse.

### Worked examples

- Try 3 Q32 — needed to lint file content immediately after a write. Picked PreToolUse ("run the linter before it completes") — but there's no file content to lint until after the write. PostToolUse was required.
- Try 3 Q20 — needed to normalize a package name after a Java file is written. Picked a custom `write_java_file` tool restricted via `tool_choice`, reasoning *"we would solve the problem at its root, instead of doing the post-tool-use which would try to fix the problem after the fact"* — treating "earlier" as inherently more root-cause, when a PostToolUse hook on the existing Write tool does the same job deterministically without adding a new tool.

## Shorter notes

**Reach for the native tool feature before a generic workaround — but the Edit tool's "right" recovery path branches on whether uniqueness is achievable.** Try 1 (Q44, Q59 — Bash + `sed` instead of Edit's `replace_all`) recurs verbatim in try 4 (Q35), try 6 (Q03, Q11), and try 9 (Q48) — five misses across four attempts, but they are **not all the same correct answer**. Don't collapse them into one reflex ("stay inside the Edit tool") — that's what caused a wrong correction after try 9. The actual decision tree, keyed on whether more context *can* disambiguate the match:

1. **Non-unique match, but more surrounding context would make it unique** (try 6 Q03, Q11; try 1 Q44/59; try 4 Q35) → stay inside the Edit tool: expand `old_string` with more surrounding lines, or Grep first to identify the correct occurrence, then expand context. Reaching for Bash/`sed` here bypasses the tool's safety guarantee for no reason — the fix was one parameter away.
2. **Non-unique match, no context *can* disambiguate it — the pattern is genuinely repeated with identical surroundings, and the job is a single one-off insertion, not a uniform change** (try 9 Q48) → Read the whole file, insert at the right spot, Write it back. `replace_all` is wrong here too — it's built for "change every occurrence the same way" (a rename, a flag flip), and using it to embed a one-off insertion into a repeated pattern corrupts every other occurrence.
3. **Never**: reach for `sed`/Bash to route around the Edit tool's non-unique-match check in either case, and never use `replace_all` as a stand-in for a targeted single-location edit.

The exam distinguishes these two shapes deliberately — read the scenario for whether "more context" is actually available before picking between "expand the search string" and "Read + Write."

**New, unconfirmed — MCP tool errors belong in the tool result's error channel, not a disguised success or external control logic.** Try 7 (`lookup_order`, `process_refund`, both this same attempt): a backend error (order not found, temporary DB failure, a business-rule refund denial) got routed around the tool result's own error mechanism — once as a success response with a status field describing the error, once as retry logic bolted on beside the tool call instead of `retryable` metadata on the result itself. Correct answer both times used the tool result's dedicated error channel (`isError: true`, structured error metadata like `errorCategory`/`isRetryable`) so the calling agent can reason about the failure directly. This is the same shape as "reach for the native tool feature before a generic workaround" above (Edit tool vs. `sed`) generalized from Claude Code's built-in tools to MCP tool design — but only one attempt's evidence so far (two misses in the same sitting, not two separate attempts), so it's logged here as a first appearance rather than promoted to its own section. Watch for a second attempt hitting this before treating it as settled.

**Resolved — watch for regression: multi-agent context misattribution.** Try 1 (Q48, Q55) and try 2 (Q14, Q59) both blamed a downstream/receiving agent for a failure that actually traced back to the coordinator not passing structured context forward (missing source URLs, incomplete task decomposition). This pattern doesn't appear in try 3, 4, or 5 — looks genuinely fixed. Logged here so a reappearance would be noticeable rather than silently written off as a one-off.

## Conflicting sub-agent outputs — resolve via the coordinator, not via the reader

Source: follow-up discussion after the try 7 mock exam (2026-08-28/29), on the "two sub-agents return conflicting figures with moderate confidence" question.

### The trap

Two sub-agents check the same metric and come back with different numbers, each self-reporting "moderate confidence." The instinct is to distrust the confidence scores (correctly — LLM self-reported confidence skews overconfident and isn't a reliable signal) and conclude from that distrust that *any* further verification step is pointless, so the safest move is to hand both numbers to the reader and let them decide. That reasoning has a true premise (confidence scores are unreliable) but draws the wrong conclusion from it (surfacing the unresolved conflict is fine).

The distrust-confidence intuition also gets misapplied a second way: reading "run a focused check" as "ask a sub-agent to check again," which sounds like retrying an already-unreliable process and hoping for a luckier roll.

### The test

Two things resolve this, independent of confidence scores entirely:

1. **"Focused check" targets the coordinator, not a sub-agent redo.** The correct action isn't re-dispatching the same kind of fuzzy retrieval that already produced two different numbers — it's the coordinator doing something more authoritative: a direct, ideally deterministic read against the primary source itself (an API call, a direct lookup) rather than another agent's general-purpose retrieval. Same shape as ["reach for the native tool feature/deterministic action before a generic workaround"](#shorter-notes) — go to source, don't re-ask.
2. **A coordinator's job is synthesis, not pass-through.** Shipping two contradictory numbers without attempting resolution isn't neutral or reader-empowering — it's the coordinator failing to do the one thing it exists for. Handing an unresolved conflict to the reader is the fallback *after* a targeted resolution attempt is inconclusive, not the first move.

### Worked example

Try 7, "two sub-agents return conflicting figures for the same metric, each with moderate confidence" — picked "include both numbers and let the reader decide" (plausible-sounding, reader stays in control) over "run a focused check that re-fetches the metric from the primary source and resolves the conflict before synthesizing" (correct). The confidence-score skepticism that drove the wrong pick was itself correct reasoning — it just doesn't bear on which answer is right, since the correct answer doesn't rely on trusting confidence scores at all.

## No option is 100% correct — rank by which failure mode is dominant

Source: follow-up discussion after the try 7 mock exam (2026-08-29), on the extraction_pipeline "materials" field question (Q58, see `../../attempts/claudecertificationguide-mockexam-try7-2026-08-28-analysis.md`).

### The trap

Multiple-choice questions on prompt engineering / extraction reliability rarely offer one flawless option and three broken ones. They offer several options that each fix *something*, and ask which is *most effective* — so holding out for the option with zero downside stalls the decision instead of resolving it. The fix isn't finding the perfect answer; it's ranking which option addresses the scenario's *dominant* failure mode.

### The test

1. **Separate the scenario's failure modes and rank them by frequency/emphasis in the prompt.** Words like "occasionally" vs. no qualifier (implying "consistently"/"often") are the ranking signal the question is handing you.
2. **For each option, ask: does this fix the dominant mode, the secondary mode, neither, or does it risk making something worse?** An option that only patches the secondary/rare failure, or that fixes nothing about the dominant one, loses even if it sounds reasonable in isolation.
3. **Pick the option that reliably fixes the dominant mode and does no active harm to the secondary one** — you don't need it to *guarantee* the secondary mode is solved, just not worsen it.

### Worked example

Try 7 Q58 — "materials" field inconsistently formatted ("cotton blend" vs. "Cotton/Polyester mix" — the dominant, unqualified pattern) and "occasionally" omitted when present (the secondary, rarer pattern). Four options: make the field required (A), switch model tier (B), temperature 0 (C), few-shot examples with standardized formats (D).

- A fixes *only* presence (forces a value to exist) — does nothing for format consistency (the dominant problem) and actively risks fabrication if the model didn't recognize the material info to begin with.
- B and C don't address either failure mode (this isn't a capability gap, and format/recognition inconsistency isn't pure sampling randomness).
- D directly and reliably fixes the dominant mode (format) by demonstrating the exact desired shape, and plausibly helps the secondary mode too (examples that show recognizing non-obvious material mentions train the model's attention on what counts as "material present") — without the fabrication risk A carries.

None of the four is airtight — D doesn't *guarantee* omission never recurs. But it's the only option that cleanly wins on the dominant failure mode and doesn't create a new risk, which is what "most effective" is actually asking for.

See also [prose-clarity-vs-examples-fix.md](prose-clarity-vs-examples-fix.md) for the sibling case of when examples are the right fix vs. a band-aid over a missing structural piece — this heuristic is the more general "how to rank among imperfect options" version of that same tension.

## Structured per-component state over raw logs/transcripts for handoffs and resumption

Source: try 9 mock exam (2026-08-30), two misses in the same attempt (`../../attempts/claudecertificationguide-mockexam-try9-ALL-2026-08-30-analysis.md`). Draft — one attempt's evidence so far, flagged for pushback.

### The trap

When a system needs to hand state from one component to another (a synthesis agent resuming after a crash, a report generator needing to trace claims back to sources), the tempting "safe" answer is to just carry forward the raw conversation log or the full accumulated text — it feels lossless, since nothing gets summarized away. But raw logs are unstructured: nothing in them is addressable, mergeable, or re-parseable without re-reading the whole thing, and free-text markers embedded in prose (an inline citation, a source-ID prefix) get silently detached the moment anything downstream reprocesses that text (paraphrases it, truncates it, merges it with other text).

### The test

Ask: **does the receiving component need to reconstruct specific facts (which claim came from where, which document was fully processed, what a given sub-task's result was) from what's handed to it?**

- If yes, and the current design hands over prose/log/transcript → the fix is to have each producing component emit a **structured artifact with its own fields** (a claim-source pair with explicit `source_url`/`date` fields, a per-document structured report the coordinator can reload) — not a bigger prose blob and not inline text markers that require re-parsing.
- If the receiving component genuinely just needs a shared narrative and isn't reconstructing anything from it — summarizing a conversation for a human, or continuing a chat where nothing needs to be individually addressed — plain narrative/log is fine (see [Missing capability vs. already-adequate piece](#missing-capability-vs-already-adequate-piece--dont-add-new-machinery-when-something-already-does-that-job)'s try 8 customer_support example, which explicitly picked narrative summarization over structured field extraction because nothing downstream needed to address the fields individually).

### Worked examples

- Try 9 (research_pipeline) — a multi-agent research pipeline crashes after 12 of 28 documents. Correct fix: each agent persists a structured report to a known location; on resume, the coordinator loads those reports and injects relevant state into agent prompts. Wrong instinct: persist the coordinator's raw conversation log (all task delegations and responses) and hand that to agents on resume — technically complete, but nothing in a raw log is addressable per-document, so resuming means re-parsing an ever-growing transcript instead of loading discrete per-document state.
- Try 9 (research_pipeline) — a synthesis agent loses track of which sources support which conclusions when merging findings, even though the upstream agents attach correct citations. Correct fix: require every subagent to output structured claim-source mappings that synthesis must preserve and merge, rather than free text. Wrong instinct: have the coordinator inject source-identifier prefixes into text before each handoff and parse them back out at report time — a text-marker workaround that's fragile to exactly the kind of reprocessing (paraphrasing, merging) the question describes, when the underlying schema-extraction gap-topic note (see [`structured-claim-source-mapping.md`](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md)) already covers why inline citations don't survive downstream transformation.

See also [`structured-claim-source-mapping.md`](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md) for the single-topic version of the second worked example, generalized here to the broader "structured state, not raw logs" shape.

## Related notes

- [prose-clarity-vs-examples-fix.md](prose-clarity-vs-examples-fix.md) — a sibling disambiguation for vague-instruction fixes specifically: prose rewrite vs. concrete examples vs. (a third case) the instruction is fine but the tool boundary itself is wrong.

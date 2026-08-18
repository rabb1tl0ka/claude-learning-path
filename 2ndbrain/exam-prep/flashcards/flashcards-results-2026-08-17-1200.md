# Flashcards session — 2026-08-17

**Scope:** `exam-prep/gap-topics` (all coverage-gap notes, `--include-gaps` implied)
**Score: 10/10**

## 1. Aggregate metrics trap

**Q:** A pipeline reports 97% overall extraction accuracy. Why is this number alone dangerous for deciding whether to automate?
**Options:** It only reflects schema validation, never semantic validation / 97% is too high to be realistic, so the number is probably fabricated / It's a volume-weighted average that can hide a low-accuracy, low-volume segment behind a high-accuracy, high-volume one / It's always calculated incorrectly by most pipelines
**Your answer:** Volume-weighted average hides segments — **Correct**
**Correct answer:** It's a volume-weighted average that can hide a low-accuracy, low-volume segment behind a high-accuracy, high-volume one

The 97% figure is a volume-weighted average — if a pipeline processes mostly one easy, high-volume document type at 99.5% accuracy alongside a small, hard, low-volume type at 60%, the easy cases dominate the denominator and the aggregate still looks great. The failing segment stays statistically invisible until it accumulates enough volume to move the average — by which point real errors have already reached production. This is why it's a gating decision (which segments get auto-approved vs. routed to humans), not just a reporting nicety.

## 2. Confidence calibration

**Q:** What does confidence calibration actually require to build a valid confidence-to-accuracy mapping?
**Options:** A second, more powerful model to cross-check the first / A labelled validation set with known ground-truth answers to compare extractions against / A larger context window to include more examples / Access to the model's internal weights
**Your answer:** Labelled ground-truth set — **Correct**
**Correct answer:** A labelled validation set with known ground-truth answers to compare extractions against

Calibration means checking what a reported confidence score actually corresponds to in real accuracy — you can only find that out by comparing extractions against known-correct answers (ground truth). This produces a per-field, often per-document-type, mapping (e.g. 0.90 confidence → 94% real accuracy on dates, but only 82% on free-text amounts). Note: this fixes what the number *means*, it doesn't make the model more accurate — that's a separate fix (prompting, context, fine-tuning).

## 3. Retry with error feedback

**Q:** What is the hard boundary on what retry-with-error-feedback can fix?
**Options:** It only works for the first retry attempt, never subsequent ones / It can only be used with schema validation, never semantic validation / It can only fix errors in numeric fields, never text fields / It cannot conjure information that is genuinely absent from the source document
**Your answer:** Can't conjure absent info — **Correct**
**Correct answer:** It cannot conjure information that is genuinely absent from the source document

Retries with specific error feedback can fix anything the model *could* have gotten right given the info already available: format mismatches, misplaced values, arithmetic errors. But if a field (e.g. a PO number) simply isn't printed anywhere on the document, no amount of retrying will produce a correct value — worse, the model may start fabricating a plausible value just to satisfy retry pressure. That's why the pipeline needs a separate detection step for "genuinely missing" vs. "extraction failed."

## 4. Risk-based human review

**Q:** What two factors define "risk" in risk-based human review allocation?
**Options:** The consequence of an error and the likelihood of an error / Document length and file format / Processing cost and processing time / Model confidence alone, nothing else
**Your answer:** Consequence x likelihood — **Correct**
**Correct answer:** The consequence of an error and the likelihood of an error

Risk = consequence of an error (financial/legal/compliance exposure) × likelihood of an error (measured accuracy from calibration/sampling data). This is why the highest-risk categories get 100% review regardless of confidence, while low-risk high-accuracy categories only need a small stratified sample — review capacity tracks risk, not a uniform percentage.

## 5. Semantic vs schema validation

**Q:** An invoice's line items are all well-typed numbers, and the stated total is also a valid number, but they sum to £450 instead of the stated £500. What catches this?
**Options:** Schema validation, since it checks all numeric fields / The model's confidence score, which would automatically flag it / Semantic (business-rule) validation, via an explicit cross-field check / Nothing catches this — it's an unavoidable extraction error
**Your answer:** Semantic validation — **Correct**
**Correct answer:** Semantic (business-rule) validation, via an explicit cross-field check

Schema validation only checks structural shape — both £450 and £500 are perfectly valid numbers, so it passes. Only an explicit cross-field business-rule check ("do line items sum to the stated total") catches the discrepancy. This is exactly why "it validated against the Pydantic model, ship it" is a false sense of security — schema passing proves well-formedness, not correctness.

## 6. Stratified sampling

**Q:** Why is it critical that stratified sampling include high-confidence, auto-approved extractions, not just low-confidence ones already under review?
**Options:** Low-confidence extractions never need sampling since they're already reviewed / High-confidence extractions are always wrong, so they need extra scrutiny / Sampling only high-confidence items is cheaper computationally / A new systematic error pattern could produce high self-reported confidence and sail through auto-approval completely unnoticed otherwise
**Your answer:** Drift can hide behind confidence — **Correct**
**Correct answer:** A new systematic error pattern could produce high self-reported confidence and sail through auto-approval completely unnoticed otherwise

Once extractions clear the calibrated threshold, they're auto-approved and nobody's looking at them anymore. If a new error pattern emerges (new vendor layout, OCR degradation) and still produces high self-reported confidence, it sails through auto-approval undetected — potentially for a long time, since the population that would normally raise alerts (human reviewers) has been bypassed entirely.

## 7. Structured claim-source mapping

**Q:** Why is inline prose citation (e.g. "According to page 12...") fragile in a multi-step pipeline?
**Options:** Downstream steps like summarization, re-templating, or merging can silently drop or detach the citation since nothing structurally binds it to the claim / It takes up too much storage space / Prose citations are not supported by the Claude API at all / It's fragile only when the source is a PDF, not other formats
**Your answer:** Nothing binds citation to claim — **Correct**
**Correct answer:** Downstream steps like summarization, re-templating, or merging can silently drop or detach the citation since nothing structurally binds it to the claim

An inline citation is just prose sitting next to the fact — nothing structurally connects them. The moment a downstream step (another LLM call, a template renderer, a truncated UI card) touches that text, the citation is exactly the kind of detail that gets silently dropped. The fix is structured claim-source pairs: the claim and its source metadata (URL, document, page, date) live in separate fields, so the source survives as data regardless of what happens to the claim text.

## 8. Tool choice forcing (structured-extraction turn)

**Q:** In a structured-extraction step where a free-text answer would break the downstream pipeline, which tool_choice approach fits best?
**Options:** `none`, so Claude responds in plain prose only / Leaving it on `auto` for every turn of the conversation / Removing all tools from the request / Forcing a specific tool (or `any`) for that turn so Claude can't just respond in prose instead
**Your answer:** Force a tool this turn — **Correct**
**Correct answer:** Forcing a specific tool (or `any`) for that turn so Claude can't just respond in prose instead

Forcing (`any`, or a named tool) exists exactly for turns where the calling code already knows a tool call is the only acceptable outcome — a structured-extraction step where free-text would break the pipeline downstream. `auto` leaves room for Claude to respond in prose instead, which is the failure mode you're trying to avoid here.

## 9. Tool choice forcing (multi-turn pattern)

**Q:** What is the common pattern for using forced tool_choice across a multi-turn conversation?
**Options:** Force a tool only on the last turn of the conversation / Never use forced tool_choice in multi-turn conversations / Force a specific tool for exactly the one turn that needs it, then switch back to `auto` afterward / Keep tool_choice forced for the entire conversation to guarantee structured output at every turn
**Your answer:** Force one turn, then auto — **Correct**
**Correct answer:** Force a specific tool for exactly the one turn that needs it, then switch back to `auto` afterward

Correct — and this is the exact pattern the note flags as "known but inconsistently applied under exam pressure" (nailed on one mock-exam question, missed on an adjacent one moments later). Force the specific tool for exactly the turn that needs guaranteed structured output, then switch back to `auto` — leaving it forced for the whole conversation would permanently block Claude from ever responding in plain text again.

## 10. Retry with error feedback (detection-first design)

**Q:** In a well-designed pipeline, which check should run first to handle the "information genuinely absent from source" case efficiently?
**Options:** An unlimited retry loop with no cap, since eventually it will succeed / A cheap deterministic/keyword presence check on the source, with the retry loop as the fallback for what it can't resolve / A second LLM call to double-check the first model's confidence / Immediately escalate every failure to a human reviewer without any automated check
**Your answer:** Cheap presence check first — **Correct**
**Correct answer:** A cheap deterministic/keyword presence check on the source, with the retry loop as the fallback for what it can't resolve

A deterministic/keyword presence check on the source is fast and high-precision when it fires, so it should run first as a cheap short-circuit for the obvious "not present" cases. But it can't be trusted alone (OCR noise, label variants, non-literal/derived fields all produce false negatives), so retry-with-error-feedback is the fallback safety net for everything the deterministic check can't resolve — not a strict either/or.

---

## Weakest topics this session

None — 10/10, all gap-topics answered correctly on first pass. All 8 coverage-gap topics now have at least one clean attempt logged in `.flashcards/history.jsonl`; `tool-choice-forcing` and `retry-with-error-feedback` each got a second question given their history of being "known but misapplied under pressure" on a real mock exam.

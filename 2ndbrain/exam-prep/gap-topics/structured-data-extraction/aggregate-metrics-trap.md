## Summary

A single blended accuracy number is one of the most dangerous metrics in a structured data extraction pipeline, precisely because it looks reassuring. "97% overall accuracy" sounds like a green light to fully automate — but that number is a volume-weighted average, and volume-weighting hides exactly the failures that matter most.

The mechanism: if a pipeline processes mostly one easy, high-volume document type (say, standard invoices at 99.5% accuracy) and a small number of hard, low-volume document types (say, scanned/handwritten receipts at 60%, or international documents at 45%), the aggregate can still land at 97%+ simply because the easy cases dominate the denominator. The teams and systems that fail on those document types are statistically invisible in the top-line number until enough of them accumulate to move the average — by which point real errors have already reached production/downstream systems.

**The fix is to always break accuracy down by segment before trusting an aggregate:**
- **By document type** (invoice vs. receipt vs. contract vs. scanned PDF vs. handwritten form)
- **By field** (a date field might extract at 98%, a free-text amount field at 80%)
- **By the cross of the two** (e.g. "amount" field specifically on scanned PDFs)

Only once every segment clears an acceptable bar should a team trust automation for that segment. This is a gating decision, not a monitoring nicety — **it determines which document types get auto-approved vs. routed to a human reviewer or a stricter model config.** A pipeline can legitimately automate the 99.5% segment while still routing 100% of the 60%-accuracy segment to humans, and the reported aggregate needs to reflect that split rather than paper over it.

This is a specific case of a more general statistical trap (Simpson's-paradox-adjacent): aggregating across heterogeneous subgroups can produce a number that is technically correct but operationally meaningless for decision-making.

## My Insights

New concept, no direct tie-in from the official courses — the closest analog is prompt evaluation (`claude-api/prompt-evaluation/prompt-evaluation.md`), which covers running evals over a test set, but the official material never gets into segmenting eval results by input subgroup the way production extraction pipelines require.

## Challenges

- Deciding what the "right" segmentation granularity is (document type? field? both crossed together?) is itself a design decision — too coarse and you're back to hiding problems, too fine and every segment has too few samples to trust the number.
- Small-sample segments are noisy: a document type with only 20 examples in an eval set can swing from 60% to 90% "accuracy" on a single extra correct/incorrect case, so segment-level numbers need their own confidence intervals or minimum sample thresholds before being acted on.

## Actions

- [x] Review this gap-topic note and add personal insights (owner: bruno)

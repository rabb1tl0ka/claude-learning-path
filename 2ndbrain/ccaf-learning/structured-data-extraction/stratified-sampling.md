## Summary

Confidence calibration (see `confidence-calibration.md`) is a one-time (or periodic) gate: it answers "is this pipeline good enough to turn on automation." Stratified sampling is the ongoing monitoring mechanism that runs *after* automation is live, and it answers a different question: "is the pipeline still good, right now, in production, on the documents actually flowing through it."

The mechanism: once extractions clear the calibrated confidence threshold, they get auto-approved and skip human review entirely — that's the whole point of automating. But that also means nobody is looking at them anymore. If the model develops a new, systematic error pattern (a subtly different invoice layout starts arriving from a new vendor, a currency symbol gets consistently misread, a schema drifts), and that error pattern happens to still produce high self-reported confidence, it will sail through auto-approval undetected — potentially for a long time, since the whole population generating alerts (human reviewers) has been bypassed.

The fix: continuously pull a **stratified random sample** of extractions — including, critically, high-confidence auto-approved ones, not just the low-confidence ones already being reviewed — across strata like document type, confidence band, and field type. A common concrete pattern is sampling 5-10% of overall volume, weighted more heavily toward higher-risk strata (e.g. 20% of financial/legal document types). Those sampled extractions get manually checked against ground truth on an ongoing basis, and any accuracy drop within a stratum is a signal that something changed (a new document format arrived, the model changed, upstream OCR degraded, etc.) before it becomes a customer-facing incident.

The key distinction to remember for exam purposes: 
- **calibration** = static/periodic, done once (or on a schedule) against a validation set, before or independent of live traffic. 
- **stratified sampling** = continuous, done against live production traffic, specifically to catch drift and novel error patterns that calibration can't anticipate because they didn't exist yet when calibration was run.

## My Insights

No direct tie-in from the official courses — the closest conceptual cousin is the idea of continuous evaluation from `prompt-evaluation.md`, but that course material is about pre-deployment eval sets, not live-traffic sampling of a deployed pipeline.

## Ideas
- Worth trying: if Bruno ever builds an automated pipeline of any kind (not just document extraction — this generalizes to any auto-approved LLM output), set up a cheap ongoing sample-and-spot-check step from day one, since retrofitting it after an incident is much more expensive than building it in.

## Challenges

- Deciding sample size and stratum weighting is a cost/risk trade-off with no single right answer — too small a sample and drift goes undetected for too long; too large and it defeats the purpose of automating (reviewer time cost creeps back up).
- Stratified sampling only catches drift that shows up in the strata being tracked — if a genuinely new stratum emerges (a document type that's never been seen before), it may not get sampled at the right rate until someone notices and adds it as its own stratum.

## Actions

- [x] Review this gap-topic note and add personal insights (owner: bruno)

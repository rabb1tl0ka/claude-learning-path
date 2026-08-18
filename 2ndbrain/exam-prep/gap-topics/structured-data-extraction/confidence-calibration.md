## Summary

When a model extracts a field and reports a confidence score alongside it (whether that's a raw self-reported "how sure am I" number, or a logprob-derived score), that number is not automatically trustworthy. Research consistently shows LLMs are prone to overconfidence when self-reporting — scores cluster in the 80-100% range regardless of whether the extraction is actually correct. Treating raw confidence as if it were a calibrated probability of correctness is a mistake many teams make when first automating a pipeline.

**Confidence calibration** is the process of finding out what a given confidence score *actually means* in terms of real-world accuracy, and it requires a labelled validation set (documents where the correct answer is already known — ground truth). The idea: **take a batch of extractions with their confidence scores, compare each to ground truth, and build a mapping from reported confidence to true accuracy**. Critically, this mapping is not uniform across the pipeline — it has to be built per field type, and often per document type too. A raw confidence of 0.90 might correspond to 94% real accuracy on a straightforward field like a date, but only 82% real accuracy on a harder field like a free-text total amount. Without calibration, a team might set a single blanket "auto-approve above 0.85" threshold that is actually far too loose for amount fields and unnecessarily strict for date fields.

**Calibration is also format/structure-dependent**: a model's confidence score, and its relationship to real accuracy, was learned (implicitly or via calibration data) on the document formats seen so far. When the pipeline encounters a new document layout or structure it hasn't been calibrated against, the confidence-to-accuracy mapping may not hold — the model can be just as overconfident (or newly underconfident) on the unseen format, and there's no way to know without checking against ground truth again. This is why calibration is a one-time gate but not a "solved forever" gate — see stratified sampling for the ongoing-monitoring half of this problem.

In practice, calibration produces per-field (and often per-document-type) thresholds: "only auto-approve extractions of field X on document type Y when confidence exceeds threshold Z," where Z was derived empirically from the validation set rather than picked as a round number.

**Calibration fixes the reported number, not the model.** Confidence calibration doesn't make the model's extractions any more accurate — it's the ML equivalent of calibrating a thermometer, not a car's steering. You're not changing the room's actual temperature (the model's real extraction accuracy); you're correcting the readout so "90% confident" reliably means "90% correct." If calibration reveals that a field's real accuracy is low, that's a diagnosis, not a fix. Actually improving the model's real accuracy on that field is a separate step, done through different means depending on what's available: prompt engineering (tighter instructions, few-shot examples of the failure cases), better context/RAG for that field, fixing upstream data quality (e.g. bad OCR), or literal model fine-tuning (retraining weights on labelled examples) if the model is fine-tunable at all — which most API-based LLM pipelines (Claude included) aren't set up for, making prompt/context/pipeline fixes the usual first lever instead.

## My Insights

Connects loosely to prompt evaluation (`claude-api/prompt-evaluation/prompt-evaluation.md`) in spirit — both are about not trusting a model's own signal without checking it against ground truth — but the official courses never cover confidence scores or threshold calibration specifically.

## Ideas
- Worth trying: for any extraction/classification pipeline built later, log raw confidence next to a small manually-labelled sample early, and plot the actual calibration curve (reliability diagram) before picking any threshold, rather than guessing a round number like 0.9.

## Challenges

- Calibration has to be re-validated whenever the document format/structure changes, or whenever the underlying model changes (different model, different version) — otherwise the thresholds silently go stale.
- Building genuinely representative labelled validation sets is expensive and easy to under-invest in, especially for rare document types/fields.

## Actions

- [x] Review this gap-topic note and add personal insights (owner: bruno)

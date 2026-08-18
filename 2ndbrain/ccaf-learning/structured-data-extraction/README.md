# Structured data extraction — how these topics fit together

These notes don't stand alone — they're stages (and cross-cutting concerns) of a single pipeline lifecycle: extract → validate → fix or route → calibrate → allocate review → monitor in production. This file maps that order so a given note's place in the whole isn't lost.

## The lifecycle, in order

1. **Extraction happens.** A model pulls fields out of a document and reports values plus (usually) a confidence score per field.

2. **Validate the extraction** — [semantic-vs-schema-validation.md](semantic-vs-schema-validation.md). Two distinct layers run here: schema validation (is `amount` a number, is `invoice_date` a valid date — purely structural) and semantic validation (does the content make business sense — e.g. do line items sum to the stated total). Schema validation passing is not evidence of correctness; only semantic validation catches the second class of error.

3. **If validation fails, decide what to do about it** — [retry-with-error-feedback.md](retry-with-error-feedback.md). Resend the document with the *specific* validation error attached (not a generic "failed" message), so the model can self-correct. This only works for genuinely fixable errors (format, arithmetic, misplaced values) — it has a hard boundary against information that's simply absent from the source, which needs its own detection step so the pipeline doesn't loop forever or fabricate a value just to pass validation.

4. **Once extractions are passing validation, find out what the confidence scores actually mean** — [confidence-calibration.md](confidence-calibration.md). Against a labelled ground-truth set, build a per-field (often per-document-type) mapping from reported confidence to real accuracy. This is a **static/periodic gate** — done once, or on a schedule, before/independent of live traffic — and it's what produces the thresholds ("auto-approve field X above confidence Z") that step 5 needs. Note: calibration only fixes what the confidence *number means* — it doesn't improve the model's actual accuracy; that requires separate work (prompting, context, fine-tuning) aimed at whatever calibration reveals as weak.

5. **Turn calibrated thresholds into a review-capacity decision** — [risk-based-human-review.md](risk-based-human-review.md). Given limited reviewer hours, allocate them by risk (consequence × likelihood of error) rather than a uniform review percentage or an all-or-nothing split. This is the step where calibration's numbers actually become a staffing/process decision — 100% review for high-risk categories, a smaller stratified sample for low-risk high-accuracy ones, dynamic queue-prioritization within what does get reviewed.

6. **Once live, keep checking that the pipeline is still good** — [stratified-sampling.md](stratified-sampling.md). Calibration (step 4) is a one-time/periodic snapshot; this is the ongoing monitoring that runs *after* automation ships, continuously sampling production traffic — including high-confidence auto-approved extractions, not just the ones already being reviewed — to catch drift (new vendor layout, OCR degradation, model change) before it silently sails through auto-approval.

## Cross-cutting concerns (apply across the whole lifecycle, not one stage)

- [aggregate-metrics-trap.md](aggregate-metrics-trap.md) — a warning that applies to *any* accuracy number produced at any stage above (calibration, review allocation, production monitoring): a single blended accuracy figure hides exactly the failing segments that matter. Always segment by document type / field / both before trusting a number enough to act on it (gate automation, size a review sample, judge whether drift monitoring shows a real problem).

- [structured-claim-source-mapping.md](structured-claim-source-mapping.md) — a design principle for *what happens downstream of extraction*, not a pipeline stage itself: once a fact/claim is extracted, preserve its source as structured data (separate fields) rather than inline prose citation, so attribution survives any later summarization, re-templating, or merging. Relevant whenever an extraction pipeline's output feeds into further synthesis rather than being consumed as-is.

## One-line map

```
extract → validate (schema, then semantic) → retry w/ error feedback (or flag as missing)
        → calibrate confidence↔accuracy → allocate review by risk → monitor live traffic (stratified sampling)

        [aggregate-metrics-trap: don't trust a blended number at any of the above]
        [structured-claim-source-mapping: if output feeds further synthesis, keep source as data, not prose]
```

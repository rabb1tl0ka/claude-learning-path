# 5.5 Human Review & Confidence Calibration

## Source
https://claudecertificationguide.com/learn/5-context-management/5-5-human-review-calibration

## Summary

### Overview & Core Challenge

This lesson addresses automated extraction and classification systems, focusing on deployment strategy for human reviewers. Central principle: the critical question isn't whether to use human review, but how to allocate limited reviewer capacity to maximize accuracy while minimizing costs. Requires understanding three interconnected concepts: confidence calibration, the dangers of aggregate metrics, and stratified sampling strategies.

### The Aggregate Metrics Trap

**The Fundamental Problem:** the most dangerous misconception in production extraction systems is relying on overall accuracy percentages to justify automation decisions. Example: a system reporting "97% overall accuracy" across all document types, which a team might use to justify automating all high-confidence extractions.

**The Hidden Failure Rates** — dramatically different performance by document category:

- **Standard invoices:** 99.5% accuracy on dates, 98.2% on amounts, 97.8% on names
- **Handwritten receipts:** 60.1% accuracy on dates, 55.3% on amounts, 71.2% on names
- **Scanned PDFs:** 72.4% accuracy on dates, 69.8% on amounts, 80.1% on names
- **International formats:** 45.2% accuracy on dates, 52.1% on amounts, 63.4% on names

The aggregate masks segments where the system fails catastrophically. Critically, the failing segments are often those with the highest business impact: handwritten receipts from field staff, international invoices from new suppliers, and scanned historical documents for compliance audits.

Mechanism: standard invoices dominate volume, creating a volume-weighted average that obscures poor performance elsewhere. Three of four document types have unacceptable accuracy, yet the overall metric appears excellent.

**The Core Rule:** "Always validate accuracy by document type AND field segment before automating. Never make automation decisions based on aggregate metrics alone."

### Stratified Random Sampling

**Definition & Purpose:** selecting representative samples from each stratum (defined categories such as document type, confidence band, or field type) for human verification. Differs from random sampling across the entire population because it ensures each subgroup receives appropriate attention proportional to its importance.

**Two Critical Functions:**
1. **Ongoing accuracy measurement:** confirms each segment maintains its validated accuracy rate over time as the system processes new documents.
2. **Novel error pattern detection:** discovers new failure modes that did not exist in the original validation set. As document distributions shift or the model encounters new formats, systematic errors may emerge that weren't present during initial training.

**The High-Confidence Extraction Problem:** counterintuitive but critical — you must sample high-confidence extractions, not just low-confidence ones. Low-confidence items are already routed to human review through normal thresholds. High-confidence items are automated. If the model develops a novel error pattern affecting high-confidence extractions, only stratified sampling of the automated (high-confidence) items will detect it. Without this, "you're flying blind on your automated extractions" — a system could develop systematic errors on a new document format that remain undetected until downstream business processes fail, causing real business damage.

### Field-Level Confidence Calibration

**Raw Confidence Scores Are Not Calibrated.** The system can output confidence scores per individual field. Example invoice extraction:
- vendorName: value "Acme Corp", confidence 0.98
- invoiceDate: value "2024-03-15", confidence 0.95
- totalAmount: value "$1,247.83", confidence 0.72
- lineItems: confidence 0.61

These raw scores represent the model's internal uncertainty measure, not actual accuracy. A model reporting 0.95 confidence might be correct 88% of the time on certain field types or 99% on others. The confidence score is relative, not absolute, and varies by field type.

**Calibration Process** requires labelled validation sets containing ground truth data:
1. Take a set of documents with known correct extractions.
2. Run the model on these documents.
3. Compare reported confidence scores to actual accuracy.
4. Build a calibration curve.

This reveals relationships like: "When the model reports 0.90 confidence on date fields, it's actually correct 94% of the time. When it reports 0.90 on amount fields, it's actually correct 82% of the time." The same numerical confidence score means entirely different things depending on field type.

**Calibrated Threshold-Based Routing.** Once calibration curves exist, they drive routing decisions:
- Fields above calibrated threshold: automated (with stratified sampling verification).
- Fields below calibrated threshold: human review.
- Fields in ambiguous zone: prioritized for human review.

### Reviewer Capacity Prioritisation

**The Limited Resource Problem:** human reviewers are expensive and represent a constrained resource. Even distribution of review capacity is explicitly identified as wasteful.

**Prioritization Principle:** route highest-uncertainty items to reviewers first. Uncertainty indicators:
- Low model confidence fields
- Extractions from ambiguous or contradictory source documents
- Document types with historically poor accuracy
- Fields where the model expresses uncertainty (multiple possible interpretations)

Spreading reviewer capacity evenly across all extractions "wastes time reviewing high-confidence items that the model handles well while leaving insufficient capacity for the uncertain items that actually need human judgement."

**Dynamic Queue Ordering:** prioritization must be dynamic, not static. As the system processes documents, the queue of items awaiting human review should be ordered by uncertainty level. When a reviewer completes one item, the next item served should be the highest-uncertainty item remaining in the queue, not simply the next item in chronological order. This ensures reviewers always focus on the items where human judgement adds the most value.

### Validation Before Automation: The Required Sequence

1. Measure accuracy by document type and field segment — not aggregate metrics.
2. Calibrate confidence scores using labelled validation sets to establish true accuracy-confidence relationships.
3. Set calibrated thresholds for automation versus human review routing.
4. Implement stratified random sampling for ongoing verification of automated extractions.
5. Only then reduce human review on segments that demonstrate consistent, validated accuracy.

Each step prevents a specific failure mode. Skipping to step 5 based on aggregate metrics is the primary trap the lesson warns against.

### Key Concept Summary

"97% aggregate accuracy can hide 40% error rates on specific document types. Validate accuracy by document type AND field segment. Calibrate confidence scores using labelled validation sets. Sample high-confidence extractions through stratified sampling. Prioritise limited reviewer capacity on the highest-uncertainty items."

### Exam Traps

1. **Aggregate Accuracy as Automation Justification** — using aggregate accuracy (e.g. 97%) to justify automating all high-confidence extractions. Reality: aggregate metrics hide per-type performance; 97% overall can mean 40% accuracy on specific document types. Validation by document type and field segment is required before automating any segment.
2. **Sampling Only Low-Confidence Extractions** — only sampling low-confidence extractions for human review. Reality: high-confidence extractions are already automated without review; only stratified random sampling of high-confidence items will detect novel error patterns affecting them.
3. **Using Raw Confidence Scores Without Calibration** — reality: raw confidence scores are not calibrated. A 0.90 confidence on dates might mean 94% actual accuracy, while 0.90 on amounts might mean only 82%. Calibration using labelled validation sets is required before confidence scores can reliably drive routing decisions.
4. **Even Distribution of Reviewer Capacity** — spreading reviewer capacity evenly across all extractions wastes time on high-confidence items the model handles well. Prioritize limited reviewer capacity on highest-uncertainty items where human judgement adds the most value.

### Practice Scenario

A system achieves 97% overall accuracy, with a proposal to automate all extractions where model confidence exceeds 95%. Correct answer identifies that aggregate accuracy may mask poor performance on specific document types or fields, and that confidence scores require calibration against labelled validation sets before use — synthesizing understanding of aggregate metric danger and calibration before using confidence scores for routing. Other options represent common misconceptions: that high-confidence extractions should never be automated (too conservative), that raising the threshold to 99% solves the problem (misses the root cause), or that model overconfidence with scale is the primary risk (confuses implementation detail with the core principle).

### Build Exercise Components

1. Create a mock extraction system outputting field-level confidence scores for at least four document types (invoices, receipts, scanned PDFs, international documents) with noticeably different confidence distributions.
2. Implement accuracy tracking broken down by document type and field segment, revealing how aggregate metrics (90%+) hide per-type failures (40-70% on specific types).
3. Build a calibration module that takes labelled validation sets and produces calibrated confidence thresholds per field type per document type, revealing how the same confidence score means different things for different field-document combinations.
4. Implement stratified random sampling selecting high-confidence extractions for ongoing verification, proportional to volume in each stratum (document type and confidence band).
5. Build a review router with dynamic priority queue ordering by uncertainty, ensuring high-uncertainty items are reviewed first as new extractions arrive, never serving items in chronological order.

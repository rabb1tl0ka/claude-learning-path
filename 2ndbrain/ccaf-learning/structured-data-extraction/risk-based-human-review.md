## Summary

Once a pipeline has confidence calibration (`confidence-calibration.md`) and ongoing stratified sampling (`stratified-sampling.md`) in place, there's still a design question left: given a limited pool of human reviewer hours, how should that capacity be allocated across everything the pipeline processes?

Two naive answers both fail in practice:
- **Uniform review percentage** (e.g. "review 10% of everything, regardless of type") wastes reviewer time on low-risk, high-accuracy document types while under-reviewing high-risk ones. Reviewing 10% of routine, 99.5%-accurate standard invoices catches almost nothing, while only reviewing 10% of a high-stakes, error-prone document type (e.g. legal contracts or payment instructions) leaves most of its errors unchecked.
- **All-or-nothing** (fully automate some types, fully manual-review others with no granularity in between) ignores that risk isn't binary — it fails to account for per-extraction confidence within a document type, and doesn't adapt as accuracy improves or degrades over time.

The better pattern is **risk-based allocation**: review capacity is assigned proportional to the *risk* of getting something wrong, not spread evenly. Risk here is a function of both the *consequence* of an error (financial/legal/compliance exposure) and the *likelihood* of an error (measured accuracy for that document type/field, from calibration and sampling data). Concretely, this typically looks like:
- **100% human review** for the highest-risk categories regardless of confidence — e.g. payment instructions, legal documents, anything where an error has outsized downstream cost.
- **A smaller, targeted stratified sample** for lower-risk, historically high-accuracy categories — enough to keep monitoring for drift, not enough to slow down throughput.
- **Dynamic/queue-based prioritization** within the reviewed set: items with the lowest confidence and/or worst historical accuracy for their stratum get reviewed first, so reviewer attention goes to what's most likely to be wrong, not processed in first-in-first-out order.

This connects the whole domain together: 
- calibration tells you what confidence really means
- stratified sampling tells you if accuracy is holding up over time
- risk-based review allocation is how those two signals actually get turned into a staffing/process decision rather than sitting as dashboards nobody acts on.

## My Insights

No direct tie-in from the official courses — closest conceptual relative is the general idea of routing/escalation based on confidence, which shows up in agentic workflow design (`claude-api/agents-and-workflows/agents-and-workflows.md`, e.g. routing workflows that send inputs down different paths based on classification) but that material never applies it specifically to human-review capacity allocation.

## Challenges

- Defining "risk" precisely (combining error likelihood and error consequence into one prioritization score) is a judgment call that has to be made by whoever owns the business process, not something a pipeline can infer on its own.
- Risk levels can shift over time (a document type that was low-risk becomes high-risk because of a regulatory change, or a type improves in accuracy and can be safely de-prioritized) — the allocation needs a review/update cadence, not a one-time assignment.

## Actions

- [x] Review this gap-topic note and add personal insights (owner: bruno)

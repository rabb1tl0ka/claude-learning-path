# Tool loading strategy: load everything vs. classify first

Standalone reference note (not a course chapter recap), requested by Bruno during
the [introducing-tool-use](introducing-tool-use.md) session — his core question:
in a production AI server, how do you know which tool schema(s) to send for an
unpredictable user message, when the course's own exercise hard-codes it?

## The two naive options

1. **Load all tools by default** — attach every tool schema to every request,
   regardless of whether the message needs one. Simple, one request per turn.
   Cost: prompt size grows with every tool you add, and an over-large tool
   list measurably increases wrong/missed tool calls once schemas start
   overlapping in purpose.
2. **Classify first, then load** — an extra upfront call (or embedding
   similarity search) picks the relevant subset of tools, then a second call
   sends only those schemas alongside the user message. Cost: doubles
   round-trips and latency for every single turn, and the classifier itself
   can misclassify.

Neither is great as a blanket default — which one wins depends entirely on
tool count.

## The pattern that actually scales: defer schemas, expose names + a lookup tool

Rather than picking between "all" and "pre-classified," expose only tool
*names* up front and give the model one additional tool — a search/lookup
tool — that fetches full schemas on demand for whichever names look relevant.
The model decides what it needs and fetches just that, in the same turn,
without a separate classification request or hand-rolled retrieval step.

This is not hypothetical — it's the exact mechanism visible in this session:
most available tools appear only by name in a `<system-reminder>`, and a
`ToolSearch` tool fetches full schemas for whichever ones become relevant.
The "classification" is just Claude reasoning about which names to search
for, which it's already good at, instead of a bespoke pre-filter you'd have
to build and maintain separately.

## Recommendation

- **Small, stable tool set (roughly under ~15-20 tools)**: load all by
  default. The overhead is negligible and the added complexity of a
  lookup/classification layer isn't worth it yet — this is exactly the right
  call for the course's own reminder-tool exercise (3 tools).
- **Large or growing tool set**: prefer the deferred/on-demand fetch pattern
  over a hand-rolled classifier call. It keeps the decision inside the same
  model call instead of adding a separate, fragile pre-classification step,
  and it scales without a second round-trip per turn.
- Avoid the naive "classify with a second full API call" approach specifically
  — it's strictly worse than the on-demand-fetch pattern for the same problem,
  since it adds latency without removing the risk of misclassification.

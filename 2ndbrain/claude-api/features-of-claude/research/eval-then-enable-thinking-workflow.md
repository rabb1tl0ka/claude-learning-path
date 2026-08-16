Research note, requested by Bruno during the [extended-thinking](../extended-thinking.md) session: he restated the recommended workflow as "run without thinking → eval → if accuracy is bad, redo with thinking enabled" and flagged that this sounds inefficient (an extra round trip to Claude), asking whether that's correct or if he's missing something.

## Answer

Your restatement of the workflow is correct — but the "inefficiency" you're worried about happens once, during **prompt development**, not on every live request in production.

- The eval-then-enable-thinking decision is made **once per prompt/use case**, while you're building and testing it (writing the prompt, running evals, deciding thinking is or isn't needed). It is not something you redo per user request at runtime.
- Once evals show a given prompt/task needs extended thinking to hit your accuracy bar, you just ship it with `thinking: true` baked in for that use case going forward. You don't dynamically try "no thinking" first and fall back to "with thinking" on every live call — that really would double cost and latency for every user, which is the bad version of this you're picturing.
- So the actual production behavior is either "this prompt always uses thinking" or "this prompt never does" — decided ahead of time via evals, not decided per-request. The one extra round trip is a one-time cost paid during development/testing, not a recurring cost paid by every user in production.

## Follow-up: what if the dev-time decision turns out wrong once you're live?

Bruno's follow-up: dev-time evals only cover the inputs you thought to test. What if your eval set looked fine without thinking, you shipped without it, but real users then hit cases where results aren't good enough — how would you even know, and what would you do about it?

Both mechanisms he guessed are correct, and this is a real gap in "decide once at dev time":

- **User feedback signals** — thumbs up/down, regenerate-response clicks, support tickets, users manually rephrasing/retrying the same question. Reactive and noisy (most users don't bother), but it's real signal and often the first thing that fires.
- **Continuous/online evals on production logs** — the more rigorous version. Standard practice: log prompt/response pairs from production (respecting whatever PII policy applies), periodically sample a subset — often stratified toward flagged/low-confidence/negative-feedback cases rather than pure random, since that's where problems concentrate — re-run your same eval rubric (or an LLM-as-judge grader) against that sample, track the score over time, and alert on regressions the same way you'd monitor any other production metric. Newly-discovered failure cases get folded back into the dev-time eval set so it stops being blind to that failure mode.

The honest framing: the eval-then-decide step happens once *per known failure mode*, but production monitoring is what surfaces failure modes you didn't know to test for — and that's when you go back and either redo the eval-with-thinking comparison, or expand the eval set and repeat the whole decision.

## Follow-up: could you decide per-request instead of once, at the server level?

Bruno's refined idea: rather than a static dev-time on/off decision, have the application itself assess *each individual user prompt* at request time and decide whether that specific prompt needs thinking — instead of relying on one blanket decision made ahead of time.

This is a legitimate pattern (sometimes called complexity-based routing or cascade/escalation routing), but it's something **you'd have to build yourself** — the workflow this chapter covers is the simple static case; the API's `thinking` parameter itself is just a boolean/budget you set per call, it doesn't have any notion of "decide for me." Ways people implement dynamic routing:

1. **Heuristic-based routing** — cheap rules based on query shape: keyword/task-type detection (math, multi-step reasoning, ambiguous open-ended questions → thinking on; simple lookups/short factual questions → thinking off), or prompt length/structure as a rough proxy for complexity.
2. **Confidence-based escalation** — call Claude without thinking first, check some signal of low confidence in the response (self-reported confidence, hedging language, unusually short/generic answers), and only retry with thinking enabled if that check fails. This is literally the eval-then-retry pattern from above, just moved from "once at dev time" to "per request at runtime" — worst case, it costs the double round-trip Bruno originally flagged as inefficient, but now it's deliberately paid only for the subset of requests that actually need it.
3. **A router/classifier step** — a small, fast, cheap model (or a short Claude call with a tiny prompt) classifies incoming query complexity before the main call runs, and that classification decides whether to enable thinking.
4. **Product/UI-level signal** — if the app already distinguishes request types (e.g. a "quick chat" vs. "deep analysis" mode toggle), just map that directly to the thinking flag — cheapest option, but only works if that signal already exists upstream.

Trade-offs to weigh: dynamic routing adapts better to the real distribution of live traffic than one static blanket decision, and can save cost on the easy majority of requests. But it adds real engineering complexity — you're building and maintaining a new component (the router/classifier/confidence-check), and that component itself is now a second thing that needs evaluation and monitoring over time (see the follow-up above) — it doesn't eliminate the need for evals, it just adds a layer that also needs them.


# NotebookLM Audio Overview prompt

For `exam-prep-survival-guide.md`. Upload that file (and, if you want the evidence to land with more weight, the try 1 through try 5 `-analysis.md` files it links out to) as sources in NotebookLM, then paste the prompt below into the Audio Overview "Customize" field.

Meant as a pre-game refresher to re-listen to right before each future mock exam — not a one-time deep dive — so it's written to end in a tight, memorable recap rather than trailing off into more evidence.

---

Focus on this as a personal coaching session, not a generic document summary. I'm Bruno, studying for the CCAF (Claude Certified Architect – Foundations) exam. This document is a survival guide of cross-cutting reasoning heuristics I've pulled out of five mock exam attempts — it's not exam content to memorize, it's about *how I reason under multiple-choice pressure* when several plausible-sounding fixes are on the table. Coach me on how to actually apply it, like you're the person walking me into the exam room and giving me my final pep talk.

Go through the guide's sections in this order, and for each one, explain the trap plainly, why it's tempting, and the concrete test/checklist to run instead — don't just read the document at me:

1. **Talking myself out of a correct first instinct** — this is the headline pattern, and the most evidenced one across all five attempts. Explain the three different pulls that cause it (a word getting misread more narrowly or loosely than it actually says; a correct general rule getting over-applied as an absolute; being drawn to the fancier-sounding option), and stress that only the first of those three is ever a legitimate reason to switch away from a first instinct. Land hard on the behavioral rule: the burden of proof is on whatever is pulling me away from my first read, not on the first read itself.
2. **Missing capability vs. already-adequate piece** — explain the single test this heuristic gives me: does the system already have a piece whose job this is? If yes, fix that piece directly. If no, add the one standard mechanism the gap actually requires. Make the distinction sharp between "adding a genuinely missing structural mechanism" (fine) and "wrapping a new layer of infrastructure around something that just needs tuning" (the trap).
3. **Tool/agent-granularity direction** — explain that this isn't a single "split" or "consolidate" default, it's a rule that points different ways depending on the scenario: split when one tool/agent covers unrelated jobs, consolidate when near-duplicates overlap in purpose. Mention the detail that splitting tools across separate MCP servers doesn't change anything the model actually sees.
4. **Hook timing depends on data availability, not pipeline position** — explain why "earlier in the pipeline" doesn't mean "more root-cause" for hook timing specifically, and give me the one-question test: does the hook need to see the result of the action, or only the request going in.
5. **The shorter notes** — briefly mention reaching for a tool's own built-in feature before a manual workaround, and that the multi-agent context-misattribution pattern from early attempts looks resolved (so I should notice if it ever comes back, not go hunting for it).

Keep the tone direct and conversational, like someone who actually knows this material talking me through it under time pressure — not reading a report aloud, and don't invent scenarios or quiz me, just explain and reinforce.

Close with a short, punchy recap I could repeat to myself in the last two minutes before starting the exam — one line per heuristic, phrased as the test to run, not the concept name.

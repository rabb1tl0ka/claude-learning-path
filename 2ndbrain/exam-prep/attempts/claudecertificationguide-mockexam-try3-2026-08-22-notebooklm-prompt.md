# NotebookLM Audio Overview prompt

For `claudecertificationguide-mockexam-try3-2026-08-22-analysis.md`. Upload that file as a source in NotebookLM, then paste the prompt below into the Audio Overview "Customize" field.

---

Focus on this as a personalized exam-coaching session, not a generic summary. I'm Bruno, and this document analyzes my mock CCAF (Claude Certified Architect – Foundations) exam attempt — I scored 826/1000 (83%, passed). Walk me through it like you're debriefing me right after the exam.

Structure it around these four failure patterns, in this order, and for each one spend real time explaining the underlying concept, not just restating my mistake:

1. **Root-cause fix vs. patch layer** — this is the biggest pattern (4 of 10 misses). Explain the difference between fixing an underspecified artifact (vague prompt criteria, a sparse tool description, a loose requirement) directly versus bolting a verification/enforcement layer on top of it. Use the Q47 and Q53 examples specifically, including that on Q53 I misread the question and missed that "accepts SQL" isn't the same as "accepts the right SQL dialect."
2. **Hook mechanics** — explain PreToolUse vs PostToolUse timing clearly: what data each one actually has access to when it fires, and why "earlier in the pipeline" doesn't mean "more root-cause" or "more robust." Use the Q20 and Q32 examples, including that I got the causality backwards on Q20 (treating a custom tool restriction as fixing things "at the root" when the PostToolUse hook was actually the deterministic guarantee).
3. **Delegation and decomposition boundaries** — explain when a coordinator agent should delegate a task to a subagent versus just handling it directly, and push back on absolute rules like "a coordinator should never do actual work." Call out that on Q59 I talked myself out of the correct answer by over-applying that exact absolute rule, right after correctly applying the *proportional* version of it on an earlier question.
4. **Confidence calibration** — briefly touch on the fact that I got several agentic-orchestration/hook questions right but flagged low confidence while doing it, which is worth being aware of even when the score doesn't show it as a weak spot.

Also mention, briefly, what I got consistently right (context window management — lost-in-the-middle, scratchpads, persistent facts — and prompt engineering), so it's not purely deficit-framed.

Keep the tone direct and conversational, like two people who actually know this material talking it through, not reading a report aloud. Don't just list the patterns — explain *why* each one is a trap, so I actually internalize it before I sit the real exam.

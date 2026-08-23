# 3.5 Iterative Refinement Techniques

## Source
https://claudecertificationguide.com/learn/3-claude-code-config/3-5-iterative-refinement

## Summary

### Core Framework: The Technique Hierarchy

The page establishes a clear pecking order for refinement approaches, ranked by effectiveness for specific scenarios.

**1. Concrete Input/Output Examples (Inconsistent Interpretation)**

Primary use case: when prose descriptions produce varying results across runs.

Mechanism: "Provide 2-3 examples showing the exact input and the exact expected output" to establish a pattern the model generalizes reliably. "The model generalises from these examples more reliably than from any prose description."

Critical rule: when facing inconsistency, the answer is never more precise prose. Exam trap explicitly warns: "More precise prose still relies on interpretation. Concrete input/output examples eliminate interpretation ambiguity. The answer to inconsistent interpretation is always examples first, not better prose."

Sufficiency principle: "Two or three concrete examples set the pattern, and the model applies it to new cases. Two or three well-chosen ones...are enough."

**2. Test-Driven Iteration (Complex Transformations)**

Primary use case: transformations with many edge cases requiring unambiguous feedback.

Test coverage requirements:
- Happy path (standard expected transformation).
- Edge cases (null values, empty inputs, boundary conditions).
- Performance requirements (if applicable).

Feedback mechanism: "The failures give concrete, unambiguous feedback about what needs fixing. There's no room for interpretation when the test output says 'Expected X, got Y.'" This eliminates prose-based corrections entirely.

**3. Interview Pattern (Unfamiliar Domains)**

Primary use case: working in domains where expertise gaps might cause missed considerations.

Mechanism: have Claude ask questions before implementing rather than prescribing solutions. The model surfaces requirements the developer wouldn't know to specify.

Expected outcome: "Claude might ask about cache invalidation strategies, TTL policies, consistency requirements, and failure modes — considerations that an expert would know to address but that you might overlook."

Critical distinction: "The interview pattern is for unfamiliar domains where the developer might miss important considerations. Concrete examples are for when the developer knows the exact transformation but the model interprets it inconsistently. Do not confuse the two — they solve different problems."

### Feedback Delivery Architecture

**Batch vs. Sequential Feedback Rule**

Batch feedback (single message) when issues interact with each other. Example: "If changing the error handling pattern also affects the logging format and the response structure, provide all three pieces of feedback in one message. The model needs to see all the interacting constraints at once to produce a coherent fix."

Sequential feedback when issues are independent: "If the naming convention issue and the indentation issue don't affect each other, fix them one at a time. Batching independent issues can confuse the model about which feedback applies to which part of the code."

Exam trap: "Not recognising when to batch vs sequence feedback. If issues interact (fixing A affects B), provide all in one message so the model sees all constraints. If issues are independent, fix sequentially. The exam tests this distinction directly."

### Workflow Pattern for Example-Based Communication

Prescribed four-step iteration sequence:

1. Observe inconsistency: describe a transformation; Claude Code produces different results each time.
2. Switch to examples: provide 2-3 concrete before/after pairs.
3. Verify generalisation: test on a new case to confirm pattern application.
4. Add edge case examples if needed: if standard cases work but edge cases fail, add examples specifically demonstrating edge case handling.

### Exam Traps (Explicit Misconceptions)

1. Choosing to refine prose descriptions when the model interprets inconsistently — directly contradicts the technique hierarchy. The solution is always examples first.
2. Not recognising the batch vs. sequence feedback distinction — the exam tests whether candidates understand when issues interact versus when they're independent.
3. Confusing the interview pattern with the examples technique — these solve fundamentally different problems (missing domain considerations vs. inconsistent interpretation).

## Note on worktree coverage (weak area flagged for extra attention)

This page was fetched twice, the second time with an explicit targeted search for the words "worktree", "steering" (in the version-control/coordination sense), "rewind", "checkpoint", "interrupt", and "coordination" anywhere on the page (including sidebars/footer). Result: **none of these appear**, except a single unrelated use of "steering" — "The exam checks that you know the specific techniques for steering Claude Code toward the right result" — which refers to steering the model's output via refinement technique, not to worktrees, session interruption, or version-control coordination.

The 3-4 (plan mode) and 3-6 (CI/CD) pages were also checked and contain no worktree content either. Across all six Domain 3 subsection pages (3.1–3.6), **git worktrees and worktree coordination are not covered at all**. This topic does not appear to live under Domain 3 on this study guide site — if it's tested, the source material would need to come from elsewhere (e.g. a Domain 4/5 page on multi-instance or long-running session workflows, or the official Claude Code docs directly).

## Overall result

**826/1000 — PASSED** (pass mark 720). 50 of 60 correct (83%). This raw score is the number closest to real CCAF readiness (per `exam-prep/CLAUDE.md`, the actual exam is scoped broadly across Claude Code/Agent SDK/API/MCP, not just the 4 official courses) — the course/gap mapping below is routing info, not a competing readiness metric.

## Domain breakdown

| Domain | Score | Note |
|---|---|---|
| D1 Agentic Architecture & Orchestration | 11/14 (79%) | |
| D2 Tool Design & MCP Integration | 8/11 (73%) | — weakest |
| D3 Claude Code Configuration & Workflows | 9/12 (75%) | |
| D4 Prompt Engineering & Structured Output | 11/12 (92%) | |
| D5 Context Management & Reliability | 11/11 (100%) | — strongest |

A Gemini transcript of Bruno narrating his live reasoning (questions 18-60, plus a revisit of flagged Q10/Q15/Q17 at the end) is available and sharpens the "why" behind several of these misses beyond what the exam's own explanations show — see inline notes below.

## Failure patterns

**1. Root-cause fix vs. patch layer (Q3, Q17, Q47, Q53)** — the single biggest pattern, 4 of 10 misses. **When the actual defect is an underspecified or ambiguous artifact** — a review category's prompt criteria, a sparse MCP tool description, a vague "integration tests" requirement in CLAUDE.md, a tool description that doesn't name a required SQL dialect — **the fix is to tighten that artifact directly**. Bruno instead reaches for a patch layer bolted on top: a second model pass (Q3), a hardcoded system-prompt override (Q17), a hook that bans the symptom instead of specifying the requirement (Q47), a SQL validation layer (Q53). Every explanation calls the chosen answer "brittle" or "doesn't scale" and says fixing the root artifact is "the correct proportionate first step." This is the CCAF exam's central theme — Claude Code problems caused by ambiguity get fixed by removing the ambiguity, not by adding process around it.

Transcript evidence sharpens two of these: on **Q47** he never voiced the correct option (tighten CLAUDE.md with concrete criteria) at all — his narration jumps straight to comparing "PostToolUse hook that rejects mock annotations" (his answer) against "a review subagent that rewrites bad tests," calling the review subagent "too expensive" and picking the hook. The specification-fix option wasn't even on his radar; he was choosing between two enforcement mechanisms, not between enforcement and specification. On **Q53** he explicitly misread the setup: "the tool description already says it accepts SQL" — dismissing the correct answer (name the Snowflake dialect explicitly in the description) because he conflated "accepts SQL" (which the description does say) with "accepts the *right* SQL dialect" (which it doesn't specify). That's a misreading, not just a wrong instinct.

> **Caveat on Q47 — the question's own standard is inconsistent, don't over-index on this one as a weak spot.** Bruno pushed back on this one during review, correctly: CLAUDE.md instructions do not *guarantee* compliance — Claude Code can still generate a mocked test against a crystal-clear CLAUDE.md rule, the same way it can still leak a credit card number against a clear redaction instruction. And this same exam elsewhere (Q10, destructive SQL migrations; Q30, credit-card redaction) explicitly *rejects* a prompt/CLAUDE.md-based fix with the reasoning "prompt instructions are probabilistic... instructions alone are insufficient," and picks a hook instead because it "provides the strongest guarantee." Q47 applies the opposite standard and calls a CLAUDE.md fix sufficient to "solve" the problem — the exam is not being consistent about when probabilistic instructions are an acceptable fix and when they aren't. The charitable read of why Bruno's chosen answer (A) still loses: its actual stated flaw isn't "hooks aren't deterministic enough," it's that banning *all* mock annotations is the wrong shape of rule — the requirement is narrower (integration tests need real DB connections; unit tests are still allowed to mock), and a blanket hook would break legitimate unit tests too. So Q47 is really testing "did you notice the fix as specified is over-broad," not "prompt vs. hook." None of the four options actually offered the strongest real answer (a hook scoped specifically to `*IntegrationTest.java`-style files checking for a real-connection pattern) — so this question requires picking the least-wrong of four flawed options, not identifying a genuinely correct one. Worth remembering as an exam-writing quirk of this particular mock (claudecertificationguide.com), not a gap in Bruno's actual understanding of hooks vs. CLAUDE.md determinism.

**2. Hook mechanics confusion (Q20, Q32)** — two related misses about *how* hooks work, not *whether* to use one. Q32: picked a PreToolUse hook to lint file content before a write completes — but PreToolUse fires before the tool runs, so there's no file content yet to lint; both requirements needed PostToolUse. Q20: reached for a custom `write_java_file` tool restricted via `tool_choice` instead of a PostToolUse hook that normalizes package names after the fact — extra tooling complexity where a hook on the existing Write tool does the same job deterministically. Both show a gap in exactly when PreToolUse vs PostToolUse fires and what each can act on.

Transcript evidence shows both misses came from confident but *inverted* causal reasoning, not blind guessing. On **Q20** he explicitly framed it as a root-cause question and got the causality backwards: "if we go with tool_choice to make sure the right java file tool actually enforces the naming convention right from the start, we would solve the problem at its root, instead of doing the post-tool-use which would try to fix the problem after the fact." He treated a custom-tool restriction as the "prevent it before it happens" option and the hook as a lesser "fix it after" patch — the opposite of the exam's framing, where the PostToolUse hook is the deterministic guarantee and the custom tool is unnecessary added complexity. On **Q32** he reasoned "it cannot be a PostToolUse [for the lint requirement]" and picked a PreToolUse hook to run the linter "before it completes" — not recognizing that a file has no content to lint until after the write happens. Both misses share the same underlying mental model error: treating "earlier in the pipeline" as inherently more root-cause/robust, when the correct trigger point depends on what data the hook actually needs to act on.

**3. Delegation/decomposition boundary judgment (Q9, Q59)** — knowing not just *when* to decompose or delegate but when *not* to. Q59: delegated both a 60-file rename *and* a trivial single-file health-check addition to subagents — missing that the trivial task should stay with the coordinator to avoid needless delegation overhead. Q9 (select-2): only got one of the two defining traits of dynamic/adaptive decomposition right, alongside correctly-identified strengths on task decomposition elsewhere (Q10, Q21 were both correct on adjacent decomposition/delegation questions), suggesting the miss is at the margins of the concept rather than a fundamental gap.

Transcript evidence on **Q59** is the clearest "talked himself out of the correct answer" moment in the whole session: he considered the correct answer directly — "why would the coordinator handle task two, single-file health check... I mean it could be C actually" — then overrode his own correct instinct with an overgeneralized absolute rule: "a coordinator should never handle work, actual work." That rule is too strong; the exam's own explanation for the adjacent Q21 (which he got right) states the coordinator *can* handle simple, well-scoped tasks directly and should only delegate when a task is large or needs specialist focus. He applied the right heuristic in Q21 but flattened it into an absolute in Q59 and picked the wrong answer as a result.

**4. Hedged confidence on agentic-orchestration/hook questions (Q20, Q22, Q31, Q38)** — a process-level pattern distinct from the content misses above. Across several D1/hook-adjacent questions, Bruno explicitly flagged low confidence even when he landed on the right answer: "80% sure" (Q22), "I'm not sure, but I picked B by elimination" (Q31), "torn" between two options (Q37/Q38). This isn't a scoring problem (most of these were correct), but it suggests the underlying mental model for orchestration/hook mechanics is less solid than the raw domain score implies — consistent with his own closing remark after submitting ("I failed three questions on the agentic architecture and orchestration, the hell") showing the D1 misses (Q9, Q20, Q59) came as a genuine surprise rather than known weak spots.

## Strength patterns

**Context window management and multi-source reliability (D5, 11/11)** — lost-in-the-middle effects, context degradation from verbose exploration, scratchpad persistence, persistent-facts extraction, silent error suppression across federated data sources, and the aggregate-metrics-trap in human review calibration were all answered correctly. This is the most consistently strong area in the exam.

**Prompt engineering and structured output (D4, 11/12)** — few-shot example construction, batch processing failure handling via `custom_id`, prompt caching layout, and multi-pass review architecture were all solid; the one miss (Q3) was the root-cause-vs-patch-layer pattern above, not a prompt-engineering concept gap itself.

## Other misses

**Q6** — confused plan mode (interactive, drives toward a change plan) with the Explore subagent (purpose-built for verbose, non-interactive codebase discovery) when asked how to understand a complex pipeline before refactoring. A single instance, no sibling pattern, but worth remembering: **plan mode is for planning *changes*, Explore is for *understanding* before you know what to change****. (Not covered in the transcript — it starts at Q18.)

**Q52** — select-3 question on fixing tool misrouting; picked "merge into one generic tool" as one of the three fixes, which the explanation explicitly rejects (the guide's own fix moves the *opposite* direction, splitting a generic tool into purpose-specific ones). This sits close to pattern 1 — reaching for a consolidation/infra move instead of the interface-design fixes (renaming overlapping tools, rewriting descriptions, checking for keyword overlap in the system prompt). The transcript's narration for this question is thin and doesn't clearly reveal which of the three picks was wrong.

**Q17 (flagged and revisited)** — this one is worth calling out separately: he flagged it mid-exam and came back to it at the end. In the revisit he was audibly circling the correct answer ("enhance the sparse description... structured output... pagination advantages over Bash") but the transcript trails off into "I don't think so, I'm going to go with this" and he submitted the system-prompt-instruction answer (D) instead — a genuine "considered the right answer, then talked himself out of it under time pressure" case, distinct from Q59's overgeneralized-rule version of the same failure mode.

## Course-topic performance

Grouped by chapter note. Routing info from the exam's tested concepts matched to existing course notes by subject matter, not by the exam's own third-party lesson labels.

| Chapter note | Score | Wrong |
|---|---|---|
| [introducing-tool-use.md](../../ccaf-learning/claude-api/tool-use-with-claude/introducing-tool-use.md) | 0/2 (0%) — weakest | Q17, Q53 |
| [agents-and-workflows.md](../../ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md) | 0/1 (0%) — weakest | Q9 |
| [automating-and-verifying-work.md](../../ccaf-learning/claude-code-in-action/automating-and-verifying-work.md) | 6/8 (75%) | Q20, Q32 |
| [prompt-engineering-techniques.md](../../ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md) | 3/4 (75%) | Q3 |
| [long-sessions-and-steering.md](../../ccaf-learning/claude-code-in-action/long-sessions-and-steering.md) | 7/9 (78%) | Q6, Q47 |
| [tool-loading-strategy.md](../../ccaf-learning/claude-api/tool-use-with-claude/research/tool-loading-strategy.md) | 4/5 (80%) | Q52 |
| [multi-turn-conversations-with-tools.md](../../ccaf-learning/claude-api/tool-use-with-claude/multi-turn-conversations-with-tools.md) | 2/2 (100%) — strongest | — |
| [sharing-and-scaling-claude-code.md](../../ccaf-learning/claude-code-in-action/sharing-and-scaling-claude-code.md) | 1/1 (100%) | — |
| [creating-custom-tools-for-claude-code.md](../../ccaf-learning/claude-api/agents-and-workflows/research/creating-custom-tools-for-claude-code.md) | 1/1 (100%) | — |
| [what-are-skills.md](../../ccaf-learning/claude-code-agent-skills/what-are-skills.md) | 1/1 (100%) | — |
| [mcp-overview.md](../../ccaf-learning/claude-api/model-context-protocol/mcp-overview.md) | 1/1 (100%) | — |
| [prompt-caching.md](../../ccaf-learning/claude-api/features-of-claude/prompt-caching.md) | 1/1 (100%) | — |

## Gap-topic performance

Informational routing only — which existing gap-topics note this maps to, not a separate readiness metric. These questions count toward the overall score like any other, since gap topics are real CCAF exam scope.

| Gap-topic note | Score | Wrong |
|---|---|---|
| [stratified-sampling.md](../../ccaf-learning/structured-data-extraction/stratified-sampling.md) | 1/1 (100%) | — |
| [aggregate-metrics-trap.md](../../ccaf-learning/structured-data-extraction/aggregate-metrics-trap.md) | 1/1 (100%) | — |
| [structured-claim-source-mapping.md](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md) | 1/1 (100%) | — |

## Next step

Run `/flashcards import exam-prep/attempts/claudecertificationguide-mockexam-try3-2026-08-22.pdf` from `2ndbrain/` to seed the missed questions into weak-spot tracking and generate a targeted study guide.

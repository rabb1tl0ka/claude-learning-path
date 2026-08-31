## Overall result

**Pass — 93.33% (56/60).**

## Domain breakdown

| Domain | Score |
|---|---|
| code_exploration | 13/15 (87%) — weakest |
| customer_support | 14/15 (93%) |
| research_pipeline | 14/15 (93%) |
| extraction_pipeline | 15/15 (100%) — strongest |

Note: the source PDF only captured the "Missed (4)" tab, not "All (60)" — so this attempt's raw data covers the 4 missed questions in full (question, answer, correct answer, explanation) but not the 56 correct ones individually. Step 2a coaching and the per-question table below are limited to the 4 missed questions as a result.

## Failure patterns

**1. Reaching for new machinery instead of the existing piece's job (2/4 misses: Q3, Q4).** Both misses added a new infrastructure layer to solve a problem the existing conversation/report-writing mechanism could already solve with better instructions. Q3 (customer_support) picked "extract and persist structured issue data into a separate context layer" when the correct fix was ordinary narrative summarization of earlier turns, preserving full history only for the active issue — the conversation already holds all three issues, it just needed smarter compaction, not a parallel data store. Q4 (research_pipeline) picked "build a confidence-calibration layer that normalizes uncertainty to 0.0-1.0 scores and weight-averages" when the correct fix was instructing the synthesis agent to structure the report with explicit well-established-vs-contested sections, preserving each source's own characterization — manufacturing a normalized score here is worse than the disease, since there's no ground-truth validation set backing that normalization (see the gap-topic note below), and it actively destroys the nuance ("varies," "±7B, 95% CI") the question says needs preserving. This is the same shape as the existing survival-guide heuristic **"Missing capability vs. already-adequate piece"** — I added both as new worked examples there rather than creating a new section, since the underlying rule ("does something already do this job? fix that, don't wrap new infrastructure around it") already covers this cleanly.

**2. Session/subagent continuity: fresh restart over resume-and-inform (2/4 misses: Q1, Q2).** Both code_exploration misses had the same shape: a session or subagent already built substantial context (an hour of work, 47 files read) before the workspace changed underneath it (files modified by a merge, functions renamed). Both times "start fresh" was picked over "resume and tell it specifically what changed." I flagged this exact question to you before running this analysis, and your pushback was fair — the reference material on "the stale context problem" explicitly says resume-and-inform is "better than nothing" but *not* a complete fix, since stale tool results can still linger in the transcript and leak into unrelated decisions. Working through it again against the existing **"No option is 100% correct — rank by dominant failure mode"** heuristic, I originally argued this resolves cleanly rather than contradicts, and added a worked example under that heuristic in the survival guide.

**Retracted (2026-08-30):** that rationalization doesn't hold up. `ccaf-guide/1-agentic-architecture/1-7-session-state-resumption.md`'s own decision matrix has a row for this *exact* scenario shape — "Resuming after modifying 3 of 50 files → Fresh start + summary" — and its "Practice Scenario Analysis" walks through the identical setup (3 changed files, named specifically) and still rules out resume as insufficient, for the precise reason the "dominant failure mode" argument tried to wave away (stale results leaking into unrelated decisions). This is a direct contradiction between claudecertificationguide.com's own teaching note and its mock-exam answer key on the same scenario, not something resolvable by ranking failure modes. The worked example built on this rationalization has been removed from `exam-prep-survival-guide.md`. Treat Q1/Q2's "correct" answers as this specific exam's answer key, not as a general rule to carry forward — the source material disagrees with itself here.

**Other misses:** none — both misses this attempt cluster into the two named patterns above; no one-off "other misses" bucket this time.

## Strength patterns

**extraction_pipeline: 15/15 (100%).** Full marks on the domain most directly tied to the confidence-calibration and structured-extraction gap-topic material — the concepts from that self-directed research note are clearly solid.

**customer_support and research_pipeline: 14/15 each (93%).** Both misses in these domains were the "reaching for new machinery" pattern above, not a knowledge gap in the domain itself — the underlying customer-support and multi-agent-synthesis concepts are otherwise well handled.

## Course/gap-topic routing (informational only)

None of the 4 missed questions map cleanly to an existing chapter note or gap-topic note by a firm match:

- **Q1, Q2 (session/subagent resume with a changed workspace)** — not course-mapped. `claude-code-in-action/long-sessions-and-steering.md` covers `/compact`, rewind, and worktrees, but not this specific "resume with informed partial invalidation" mechanic — a loose thematic connection at best, not a real match. Not gap-mapped either; no existing `ccaf-learning/` topic dir covers session-resume/stale-context specifically. This looks like a genuine coverage gap worth a future gap-topic note (that's `/flashcards import`'s call to make, not this skill's).
- **Q3 (customer_support context management)** — same verdict: loosely thematically related to `long-sessions-and-steering.md`'s `/compact` coverage, but not a firm match; not gap-mapped.
- **Q4 (research_pipeline uncertainty synthesis)** — **gap-mapped** to [`ccaf-learning/structured-data-extraction/confidence-calibration.md`](../../ccaf-learning/structured-data-extraction/confidence-calibration.md). That note already establishes that raw confidence scores need ground-truth calibration to mean anything; this question is a good real-exam illustration of the flip side — building an ad hoc normalization/weighting scheme with no validation data is worse than just preserving each source's own characterization. Worth a quick add to that note's "My Insights" if you want the connection captured there too.

## Survival guide updates (draft — please push back)

- Added Q3 and Q4 as new worked examples under **"Missing capability vs. already-adequate piece"** in `../study-guide-weak-spots/notes/exam-prep-survival-guide.md`.
- Added Q1 and Q2 as a new worked example under **"No option is 100% correct — rank by which failure mode is dominant"** in the same file, directly addressing the objection you raised before this analysis ran (that resume-and-inform isn't actually risk-free per the source material) by framing it as "wins on the dominant failure mode," not "risk-free."

Both are drafts — flagging per the usual convention, since the heuristics that have held up best came from you pushing back on a first draft.

## Answer-by-answer coaching

**Q1 (code_exploration).** Your codebase exploration tool stores session IDs to allow engineers to continue investigations across work sessions. An engineer spent an hour yesterday analyzing a legacy authentication module, building context about its architecture and dependencies. They want to continue today. The session ID is valid, but version control shows 3 of the 12 files the agent previously read were modified overnight by a teammate's merge. What approach best balances efficiency and accuracy?
- Your answer: B. Start a fresh session to ensure the agent works with current codebase state without stale assumptions.
- Correct: **C. Resume the session and inform the agent which specific files changed for targeted re-analysis.**
- Coaching: Resuming preserves the hour of built architecture/dependency context on the 9 files that *didn't* change, and directly targets the only named risk (the 3 that did) with specific re-analysis — the efficiency loss of a full fresh session buys accuracy on a problem the targeted approach already solves. B isn't wrong because "fresh is bad" in general; it's wrong because it pays the full cost of discarding everything to fix a problem that's actually narrow and nameable. See the "No option is 100% correct — rank by dominant failure mode" survival-guide heuristic — the dominant risk here is narrow (3 files), so the option that directly targets it wins over the option that nukes everything.

**Q2 (code_exploration).** An engineer's exploration subagent spent 30 minutes analyzing a legacy payment system, reading 47 files and documenting data flows. The session was interrupted when the engineer's connection dropped. While away, a teammate merged a PR that renamed two utility functions. The engineer wants to continue the same exploration. What's the most effective approach?
- Your answer: C. Launch a fresh subagent with a summary of prior findings.
- Correct: **D. Resume the subagent from its previous transcript and inform it about the renamed functions.**
- Coaching: Same shape as Q1, at larger scale (47 files vs. 12) — the more context already built, the more expensive a fresh restart is, and a summary-fed fresh subagent still loses the actual transcript detail (which files led to which conclusion, not just the conclusions themselves). Resuming the transcript directly, with a pointed note about the two renamed functions, keeps all 47 files' worth of documented data flow and patches exactly the one thing that changed.

**Q3 (customer_support).** A customer raises three separate issues during one session: a refund inquiry (turns 1-15), a subscription question (turns 16-30), and a payment method update (turns 31-45). At turn 48, the customer asks "What happened with my refund?" The conversation is approaching context limits. What strategy best maintains the agent's ability to address all issues throughout the session?
- Your answer: A. Extract and persist structured issue data (order IDs, amounts, statuses) into a separate context layer.
- Correct: **C. Summarize earlier turns into a narrative description, preserving full message history only for the active issue.**
- Coaching: The conversation itself already contains everything needed for all three issues — the fix the question wants is smarter compaction (narrative summary for the two closed/inactive issues, full detail kept only where it's actively being used), not a second data store that duplicates and diverges from the conversation. A builds new infrastructure to hold information the context window could already represent more cheaply through summarization. This is the "reaching for new machinery instead of the existing piece's job" pattern — ask "does something already do this job?" before adding a parallel structure.

**Q4 (research_pipeline).** Production reviews reveal inconsistent handling of uncertainty in final reports. Sometimes conflicting subagent findings are synthesized into a single confident statement (losing nuance), while other times reports over-hedge with excessive qualifications (becoming unhelpful). When the web search agent returns "industry analysts estimate $50B market size (methodology varies)" and the document analysis agent returns "peer-reviewed study estimates $35B (±7B, 95% CI)," the coordinator either picks one arbitrarily or produces vague statements like "the market may be $35B-$50B depending on factors." What systematic approach best addresses this?
- Your answer: B. Implement a confidence calibration layer that normalizes subagent uncertainty expressions to standardized probability scores (0.0-1.0), then weight-average findings by their calibrated confidence.
- Correct: **C. Instruct the synthesis agent to structure reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context.**
- Coaching: A confidence-calibration layer only means something when it's built against a labelled ground-truth validation set (see `ccaf-learning/structured-data-extraction/confidence-calibration.md`) — there's no such data here, so normalizing "methodology varies" and "±7B, 95% CI" into invented probability scores and weight-averaging them manufactures false precision and actively destroys exactly the methodological context the question says needs preserving. The correct fix doesn't need calibration machinery at all — it just needs the synthesis agent's own report structure to keep each source's characterization intact and separate the well-established from the contested, which is a writing/structuring instruction, not a new scoring system.

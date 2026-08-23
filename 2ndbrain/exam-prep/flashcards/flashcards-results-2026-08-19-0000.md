# Flashcards results — 2026-08-19

**Score: 7/10**

Follow-up quiz on the 10 topics that had import-only history but no cached quiz card until this session (coordinator-context-passing, role-scoped-subagent-splitting, few-shot-vs-tool-granularity, prompt-clarity-vs-verification-patch, concrete-examples-vs-vague-prose, prompt-keyword-overlap, hooks-vs-claude-md-enforcement, hook-lifecycle-events, stale-context-recovery, worktree-coordination).

## 1. Worktree coordination — Claude Code in Action / Long-running sessions, steering, and CLAUDE.md configuration

**Q:** Two Claude Code instances are running in separate git worktrees — Instance A extracting the payment service, Instance B extracting the inventory service — and both need to modify the same shared OrderService.java. How should the team coordinate this?

**Your answer:** No coordination needed — **Wrong** (correct: Sequence the edits)

Worktrees isolate the *working directory* for each session, not the underlying git history the branches will eventually merge into. Both instances still edit the same file conceptually — they just don't see each other's uncommitted changes while working. That isolation gives a false sense of safety: once both branches try to merge `OrderService.java` back, you get two independent, potentially contradictory modifications colliding. The correct pattern is to sequence it — one instance finishes and merges first, the other rebases onto that updated state before touching the file itself.

## 2. Prompt clarity vs. verification patch — Building with the Claude API / Prompt Engineering Techniques

**Q:** A CI/CD code review pipeline has a 40% false positive rate on documentation-mismatch findings, causing developers to ignore all review categories. What is the most effective fix?

**Your answer:** Rewrite the criterion — **Correct**

A 40% false-positive rate on one category means the criterion itself is under-specified. Fixing that criterion removes the noise at its source. Adding a verification pass or dropping the category entirely both dodge the actual defect: one adds cost without fixing the ambiguity, the other throws away a review category instead of making it work.

## 3. Prompt keyword overlap — Building with the Claude API / Prompt Engineering Techniques

**Q:** A system prompt defines two review categories: 'Check for security vulnerabilities in each function' and 'Check for performance issues in each loop.' The model frequently calls the wrong tool. What is the root cause and best fix?

**Your answer:** Overlapping phrasing — **Correct**

Both instructions describe scanning code structures with similar verbs and framing — no sharp semantic boundary to key off of. Rewriting with distinct, non-overlapping keywords removes the ambiguity at the source; XML tags add structure but not semantic distinction, and more examples or lower temperature just make the same confusion more consistent, not correct.

## 4. Few-shot vs. tool granularity — Building with the Claude API / Prompt Engineering Techniques

**Q:** A tool called analyze_content with the description 'Analyses content from various sources' is used indiscriminately for web scraping, document parsing, and code analysis, leading to poor results for each. What is the most effective fix?

**Your answer:** Rewrite description — **Wrong** (correct: Split into purpose-specific tools)

This is the granularity problem again — one tool doing three unrelated jobs badly. No description, however detailed, fixes a tool whose actual scope is too broad; it needs to be split into `scrape_web`, `parse_document`, `analyze_code`, each owning one job. Third time this exact pattern has tripped you up across sessions (`tool-choice/tool-granularity confusion`, `few-shot-vs-tool-granularity`): when a tool does too many unrelated things, the fix is splitting responsibilities, not documenting the mess better.

## 5. Role-scoped subagent splitting — Building with the Claude API / Agents and workflows

**Q:** A code review agent equipped with 18 tools has slow reviews and frequently selects the wrong tool. What is the most effective architectural change?

**Your answer:** Split into role-scoped subagents — **Correct**

18 tools on one agent is a choice-set problem — every call has to sift through 18 options, regardless of documentation quality. Splitting into role-scoped subagents (security, style, performance), each with only relevant tools, shrinks the choice set per agent. Better descriptions or one mega-tool don't reduce the actual decision space.

## 6. Stale context recovery — Claude Code in Action / Long-running sessions, steering, and CLAUDE.md configuration

**Q:** A Claude Code agent has been working on a feature branch for 45 minutes. Tool results from 40 minutes ago are now stale because a colleague pushed changes to those files. What is the best recovery strategy?

**Your answer:** Fresh session + summary — **Correct**

Re-reading in the same session doesn't remove the stale copy already in context — the model now has both fresh and stale versions of the same file to potentially reference. A fresh session seeded with a distilled summary of valuable findings clears the stale context entirely, then a clean re-read gives it only current file state.

## 7. Hook lifecycle events — Claude Code in Action / Automating and verifying work

**Q:** A team wants to archive the full conversation transcript to a log file every time Claude Code runs /compact. Which hook configuration achieves this?

**Your answer:** PreCompact hook — **Correct**

Compaction is its own dedicated lifecycle moment, not a tool call — `PreCompact` fires immediately before it happens. There's no built-in "Compact" tool to hook `PreToolUse` onto, and neither `PostToolUse` nor `Stop` fire at the right moment.

## 8. Hooks vs. CLAUDE.md enforcement — Claude Code in Action / Automating and verifying work

**Q:** A developer-tool agent has a PreToolUse hook (100% enforced) and a system-prompt backup instruction (88% followed). A senior engineer wants all guardrails converted to hooks for consistency. What is the correct assessment?

**Your answer:** Convert backup rule — **Correct**

The path-restriction rule is already a hook at 100% — no issue there. The backup rule is a hard requirement ("always") only followed 88% of the time — that gap between "must always happen" and "usually happens" is exactly what a hook closes. Not "convert everything to hooks for consistency" (over-applies enforcement) — matching the mechanism to a specific rule that's both hard-required and currently unreliable.

## 9. Coordinator context-passing — Building with the Claude API / Agents and workflows

**Q:** A synthesis agent produces a report with several claims that have no source attribution. The web search and document analysis subagents are working correctly and returning well-sourced results. What is the most likely root cause?

**Your answer:** Synthesis prompt gap — **Wrong** (correct: Coordinator dropped context)

The subagents are confirmed well-sourced, so the source data existed — it just never reached the synthesis agent. In a hub-and-spoke architecture, the coordinator is the only thing that sees every subagent's output; missing source attribution downstream is almost always the coordinator failing to forward that structured context, not a missing instruction in the receiving agent's own prompt.

This is the strongest recurring miss in your history — the exact same question, missed on try 1, try 2, and now again (4 times total). The pattern: looking at the receiving agent's prompt when the actual fault sits one level up, in what the coordinator chose to hand off.

## 10. Concrete examples vs. vague prose — Building with the Claude API / Prompt Engineering Techniques

**Q:** A code review prompt classifies severity using prose descriptions like 'critical means the code is dangerous,' and judgments are inconsistent. What is the most effective improvement?

**Your answer:** Concrete labeled examples — **Correct**

"Dangerous" is subjective and unanchored — concrete labeled examples for each severity level give Claude something to pattern-match against instead of interpreting an adjective. More adjectives just describes the same vague concept in more words; a second pass or lower temperature don't touch the underlying ambiguity.

## Weakest topics this session

- **coordinator-context-passing** — missed a 4th time across two mock-exam attempts and two flashcard sessions; the single strongest, most persistent miss in your entire history. Worth deliberately drilling until "check the coordinator's handoff first" becomes automatic.
- **few-shot-vs-tool-granularity** — same reflexive "improve the description" default that's now cost points 3 times across different sessions, always on the same underlying tool-granularity concept.
- **worktree-coordination** — new topic, missed on first exposure; the "worktrees isolate everything" assumption doesn't extend to files both branches will merge back into.

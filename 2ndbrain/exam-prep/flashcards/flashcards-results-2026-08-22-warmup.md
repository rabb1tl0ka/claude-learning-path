# Flashcards results — 2026-08-22 (warmup, weak-spots focus)

**Score: 19/20**

Scope: weak-spot-weighted questions pulled from Agents and Workflows, Prompt Engineering Techniques,
Introducing Tool Use, Automating and Verifying Work, and Long-Running Sessions (the chapters with
existing wrong-rate history — coverage-gap topics excluded since `--include-gaps` wasn't requested).

---

### Q1 — coordinator-context-passing (Agents and workflows)
**Q:** A synthesis agent produces a report with several claims that have no source attribution. The web search and document analysis subagents are working correctly and returning well-sourced results. What is the most likely root cause?
**Options:** Coordinator dropped context / Synthesis prompt missing cite instr / Web search not actually sourced / Normal variance
**Your answer:** Coordinator dropped context — **Correct**
**Correct answer:** The coordinator failed to forward the subagents' sourced findings as context into the synthesis agent's input.

Each subagent can do its job perfectly, but if the coordinator doesn't explicitly pipe subagent A's output into subagent B's (or the synthesizer's) input, that information just evaporates between hops. It's not a prompting problem on the synthesis agent's end — no amount of "please cite sources" instruction fixes missing data. Always check the coordinator's data-flow wiring first when downstream agents seem to be missing upstream findings.

---

### Q2 — coordinator-context-passing (Agents and workflows)
**Q:** A customer-support coordinator routes a ticket to a billing subagent (confirms account tier) then to a technical subagent (troubleshoots a feature outage). The technical subagent ignores that the customer's tier doesn't include that feature at all. Root cause?
**Options:** Billing misreported tier / Coordinator didn't pass tier info / Expected behavior / Technical prompt missing instr
**Your answer:** Coordinator didn't pass tier info — **Correct**
**Correct answer:** The coordinator never passed the billing subagent's account-tier finding into the technical subagent's input.

Same pattern as Q1, different scenario. This topic sits at 36% wrong-rate historically over 11 attempts — worth internalizing as a reflex: whenever a downstream agent seems to be "missing" info another agent already found, check the coordinator's wiring before touching either subagent's prompt.

---

### Q3 — role-scoped-subagent-splitting (Agents and workflows)
**Q:** A code review agent equipped with 18 tools has slow reviews and frequently selects the wrong tool. What is the most effective architectural change?
**Options:** Lower temperature / Split into role-scoped subagents / Better descriptions / One generic tool
**Your answer:** Split into role-scoped subagents — **Correct**
**Correct answer:** Split the single agent into role-scoped subagents (security, style, performance), each with a small tool subset relevant to its role.

Past a certain tool count, a single agent's tool-selection accuracy degrades regardless of how good the descriptions are, because it's choosing from too large and semantically overlapping a set at once. Narrowing each subagent's toolbox to what its role actually needs fixes both speed and accuracy — better descriptions only help at the margins, they don't fix a fundamentally too-broad selection problem.

---

### Q4 — coordinator-responsibilities (Agents and workflows)
**Q:** A user asks a hub-and-spoke research system (search, analysis, synthesis subagents) a simple factual question. Per the "dynamic subagent selection" responsibility, what should the coordinator do?
**Options:** Always invoke all three / Synthesis decides first / Random subset / Select only what's needed
**Your answer:** Select only what's needed — **Correct**
**Correct answer:** Dynamically select just the subagent(s) actually needed (e.g. only web search) rather than always routing through the full pipeline.

One of the coordinator's core jobs is deciding *which* subagents a given request actually needs, not just passing context between them. Always-invoke-everything wastes latency/cost on trivial requests; dynamic selection is what makes hub-and-spoke actually efficient rather than just a fixed pipeline in disguise.

---

### Q5 — narrow-decomposition-failure-tracing (Agents and workflows)
**Q:** A multi-agent research system assigned "renewable energy technologies" decomposes this into only "solar panel efficiency" and "wind turbine design." Each subagent produces thorough research. The final report is comprehensive on solar and wind but missing geothermal, tidal, biomass, and nuclear fusion entirely. Root cause?
**Options:** Web search too narrow / Synthesis dropped content / Decomposition too narrow / No shared memory
**Your answer:** Decomposition too narrow — **Correct**
**Correct answer:** The coordinator's task decomposition was too narrow — it never assigned those subtopics to any subagent.

When subagents each do great work but the *whole* is still incomplete, the bug is usually upstream at the coordinator's decomposition step, not in any individual subagent's execution. Tell-tale sign: every subagent that *did* run produced high-quality output — if the failure were in execution, you'd expect sloppy output somewhere, not clean gaps.

---

### Q6 — few-shot-vs-tool-granularity (Prompt Engineering Techniques)
**Q:** A tool called `analyze_content` with the description "Analyses content from various sources" is used indiscriminately for web scraping, document parsing, and code analysis, leading to poor results for each. Most effective fix?
**Options:** Split into purpose-specific tools / More detailed description / Add few-shot examples / Lower temperature
**Your answer:** Split into purpose-specific tools — **Correct**
**Correct answer:** Split `analyze_content` into purpose-specific tools (`scrape_web`, `parse_document`, `analyze_code`), each scoped to one job.

This is your all-time worst topic (80% wrong-rate historically). The trap is reaching for few-shot examples to patch a granularity problem: examples teach a model *how* to use a correctly-scoped tool, but can't fix a tool that's fundamentally trying to be three different tools at once. When one tool serves unrelated jobs badly, the fix is architectural (split it), not prompting (explain it better).

---

### Q7 — root-cause-to-mechanism matching (Prompt Engineering Techniques)
**Q:** A rubric criterion is written as vague prose (e.g. "flag severe issues") and Claude's judgments are inconsistent across similar cases. What's the correct fix?
**Options:** Second re-check pass / More guideline bullets / Concrete examples per level / Lower temperature
**Your answer:** More guideline bullets — **Wrong**
**Correct answer:** Replace the vague prose description with concrete examples showing what counts as each severity level.

Adding more prose bullets is the reflexive-but-wrong move: it feels thorough, but it's still prose describing a fuzzy concept in more words — **it doesn't give the model a concrete boundary to pattern-match against**. Concrete labeled examples work because they let Claude generalize from instances rather than interpret abstract adjectives like "severe," which different reasoning paths can weigh differently each time. Classic "show, don't tell": examples anchor judgment far more reliably than elaborated description.

---

### Q8 — prompt-clarity-vs-verification-patch (Prompt Engineering Techniques)
**Q:** A CI/CD code review pipeline has a 40% false positive rate on documentation-mismatch findings, causing developers to ignore all review categories. Most effective fix?
**Options:** Remove the category / Add verification pass / Lower temperature / Rewrite the criterion
**Your answer:** Rewrite the criterion — **Correct**
**Correct answer:** Rewrite the documentation-mismatch criterion so it's specific enough to stop over-flagging, rather than patching around it downstream.

The tempting-but-wrong move is bolting on a second verification pass to catch what the first pass gets wrong. That adds cost and latency while leaving the root cause (an underspecified criterion) intact. If the prompt itself is ambiguous, fix the prompt — don't build downstream infrastructure to compensate for upstream vagueness.

---

### Q9 — concrete-examples-vs-vague-prose (Prompt Engineering Techniques)
**Q:** A code review prompt classifies severity using prose descriptions like "critical means the code is dangerous," and judgments are inconsistent. Most effective improvement?
**Options:** Second re-score pass / More adjectives / Lower temperature / Labeled concrete examples
**Your answer:** Labeled concrete examples — **Correct**
**Correct answer:** Replace the vague prose severity definitions with concrete labeled examples of what counts as each severity level.

Caught this right after missing the near-identical Q7 — same underlying pattern, different wording. This repeats because it's genuinely one of the highest-leverage prompt-engineering moves: whenever a classification/severity judgment is inconsistent, suspect vague prose definitions first, and fix with concrete examples, not more description or more passes.

---

### Q10 — prompt-keyword-overlap (Prompt Engineering Techniques)
**Q:** A system prompt defines two review categories: "Check for security vulnerabilities in each function" and "Check for performance issues in each loop." The model frequently calls the wrong tool. Root cause and best fix?
**Options:** Need few-shot examples / Overlapping phrasing / Need XML tags / Lower temperature
**Your answer:** Overlapping phrasing — **Correct**
**Correct answer:** The two instructions use overlapping/similar phrasing that doesn't clearly map to distinct tools — rewrite them with distinct, non-overlapping keywords tied to each tool's actual scope.

Security-vs-performance sounds semantically distinct, but the trap is realizing overlap can be structural (both say "check for X issues in each Y"), not just topical — the model pattern-matches on phrasing shape as much as meaning. XML tags organize structure but don't resolve semantic/keyword ambiguity between two similarly-shaped instructions.

---

### Q11 — tool-choice/tool-granularity confusion (Introducing Tool Use)
**Q:** Two tools overlap in what they can do (e.g. one sets a reminder for "a date," another for "a duration from now"), and Claude picks the wrong one under time pressure. What's actually wrong here?
**Options:** Need few-shot example / Descriptions too short / Overlapping granularity / Need retry verification
**Your answer:** Overlapping granularity — **Correct**
**Correct answer:** The tools have overlapping responsibility/granularity, not just unclear wording — they should be redesigned so each owns a distinct, non-overlapping job.

Same class of problem as the `analyze_content` question (Q6): when two tools' *jobs* genuinely overlap, no amount of better wording, examples, or retry logic fixes it, because there's no clean line for Claude to draw. The fix is always architectural — redesign the tool boundaries themselves.

---

### Q12 — tool description quality (Introducing Tool Use)
**Q:** Claude is repeatedly calling the wrong tool out of two similarly-named tools in a set. Per the course's guidance on tool descriptions, what's the most direct fix?
**Options:** Second double-check call / More few-shot examples / Lower temperature / Rewrite for distinctness
**Your answer:** Rewrite for distinctness — **Correct**
**Correct answer:** Rewrite both descriptions to be more distinct about what each tool does, when to use it, and what it returns.

Contrast with Q11: there, the tools *genuinely overlapped* in job/granularity, so no description rewrite could fix it. Here, the tools are presumably distinct in purpose but just poorly *described* (similarly-named, vague), so sharpening descriptions is correct and sufficient. Same surface symptom ("wrong tool picked"), different root cause — distinguishing "bad description" from "bad tool boundaries" is exactly the judgment call the exam likes to test.

---

### Q13 — multi-block assistant messages (Introducing Tool Use)
**Q:** When Claude decides to use a tool, what does the assistant message actually contain?
**Options:** Only a tool_use block / Plain-text string / Multiple content blocks possible / A tool_result block
**Your answer:** Multiple content blocks possible — **Correct**
**Correct answer:** Potentially multiple content blocks — e.g. a text block plus a `tool_use` block with the tool's id, name, and input.

Mechanical API-shape fact worth being solid on: Claude can (and often does) emit reasoning/commentary text *and* a tool call in the same assistant turn — the response `content` array holds a list of blocks, not one exclusive type. Handle both block types in that one message, don't assume either-or.

---

### Q14 — hooks-vs-claude-md-enforcement (Automating and verifying work)
**Q:** A developer-tool agent has a PreToolUse hook blocking file writes outside the project directory (enforced 100%), and a system-prompt instruction "always create a backup before overwriting" (followed 88% of the time). A senior engineer wants all guardrails converted to hooks. Correct assessment?
**Options:** 88% is fine / Hooks only for paths / Reasonable to convert / Remove backup rule
**Your answer:** Reasonable to convert — **Correct**
**Correct answer:** Converting the backup rule to a hook is reasonable since it's a hard requirement being followed inconsistently — hooks guarantee enforcement where prompt instructions only request it.

Key distinction: prompts/CLAUDE.md are instructions the model *follows* (probabilistically, hence 88% not 100%), while hooks are code that *runs* deterministically and can outright block an action. If something is a hard "always" requirement, a hook is the only mechanism that actually guarantees it.

---

### Q15 — hook-lifecycle-events (Automating and verifying work)
**Q:** A team wants to archive the full conversation transcript to a log file every time Claude Code runs /compact, so context lost during compaction can be reviewed later. Which hook configuration achieves this?
**Options:** PreToolUse on 'Compact' tool / PostToolUse watching keyword / Stop hook / PreCompact hook
**Your answer:** PreCompact hook — **Correct**
**Correct answer:** A `PreCompact` hook, which fires immediately before compaction happens.

Compaction isn't a tool call, so `PreToolUse`/`PostToolUse` (which fire around tool invocations) don't apply here at all — that's the trap in the first two options. `PreCompact` is a dedicated lifecycle event specifically for this moment, letting you capture/archive state right before context gets compressed away.

---

### Q16 — risk-matched enforcement (hooks vs prompts) (Automating and verifying work)
**Q:** A team wants to make sure Claude never force-pushes to a shared branch. Per the CLAUDE.md/Skills/Hooks three-way split, which mechanism actually guarantees this, and why not just add a CLAUDE.md rule?
**Options:** CLAUDE.md is enough / A hook / A Skill / Model judgment suffices
**Your answer:** A hook — **Correct**
**Correct answer:** A hook — because CLAUDE.md and skills are instructions Claude *follows* and can still be skipped/forgotten, while a hook is code that *runs* and can outright block the action.

Same underlying principle as Q14, applied here — "never" requirements with real consequences (force-pushing to shared branches, deleting prod data) belong in hooks, not prose instructions, no matter how high-priority or well-worded the instruction is.

---

### Q17 — worktree-coordination (Long-running sessions, steering, and CLAUDE.md configuration)
**Q:** Two Claude Code instances are running in separate git worktrees — Instance A extracting the payment service, Instance B extracting the inventory service — and both need to modify the same shared OrderService.java. How should the team coordinate this?
**Options:** No coordination needed / Separate copies, merge programmatically / Edit independently, resolve after / Sequence: one merges, other rebases
**Your answer:** Sequence: one merges, other rebases — **Correct**
**Correct answer:** Sequence it: one instance completes its edit and merges first, then the other rebases onto the updated state before making its own changes.

Worktrees isolate the *working directory*, not the underlying shared history or shared files' logical content. If two instances both touch the same file, that's a genuine coordination problem worktrees don't solve for you; letting both edit independently and resolving conflicts after works but wastes effort on work likely to be discarded/reworked — sequencing avoids that waste entirely.

---

### Q18 — stale-context-recovery (Long-running sessions, steering, and CLAUDE.md configuration)
**Q:** A Claude Code agent has been working on a feature branch for 45 minutes. Tool results from 40 minutes ago (file reads) are now stale because a colleague pushed changes to those files, and the agent is reasoning from the outdated contents. Best recovery strategy?
**Options:** Ignore, model deprioritizes / Stay, ask to re-read / Restart with --continue / Fresh session + summary
**Your answer:** Fresh session + summary — **Correct**
**Correct answer:** Start a fresh session seeded with a summary of the valuable findings so far, then re-read the changed files from that clean state.

Asking the agent to "just re-read" doesn't remove the stale reads still sitting in context; the model can still weight the old contents, especially if they were reasoned over extensively. A clean session with a distilled summary avoids that contamination while preserving useful work already done — same underlying idea as `/compact` or manual context resets, just triggered by staleness rather than context-window pressure.

---

### Q19 — effective-claude-md-rules (Long-running sessions, steering, and CLAUDE.md configuration)
**Q:** Which CLAUDE.md rule is written effectively, per the guidance that rules should be specific and checkable?
**Options:** "Be careful with API routes" / "Follow best practices for API design" / "Put new API routes in src/api/handlers, one per file" / "Don't use default exports" (no alternative named)
**Your answer:** "Put new API routes in src/api/handlers, one per file" — **Correct**
**Correct answer:** Same.

Specific (names an exact path) and checkable (verify compliance by looking at the file structure). The others are all vague-prose failures — "be careful," "follow best practices," "don't use X" without naming the alternative all leave room for interpretation, which is what makes a CLAUDE.md rule unreliable. Same root lesson as the vague-rubric prompt-engineering questions, applied to CLAUDE.md instead of a prompt.

---

### Q20 — claude-md-hierarchy (Long-running sessions, steering, and CLAUDE.md configuration)
**Q:** A consultancy works across 12 client projects. Each developer has personal preferences (editor keybindings), the firm has firm-wide coding standards, and each client project has its own conventions. What's the correct configuration architecture?
**Options:** Everything in project root / User file for everything + imports / Firm standards at user level / User/project/dir split
**Your answer:** User/project/dir split — **Correct**
**Correct answer:** User `~/.claude/CLAUDE.md` for personal preferences; project CLAUDE.md for firm-wide standards and client conventions; directory-level CLAUDE.md for subsystem rules.

Maps each rule to the scope it actually applies at: personal (user-level, loads everywhere for you specifically), organizational/client (project-level, per-repo), subsystem (directory-level, path-specific). Putting firm-wide standards at the user level would only apply to *your* sessions, not every developer's — CLAUDE.md configuration should live where it needs to be shared, not where it's convenient for one person.

---

## Weakest topics this session

- **root-cause-to-mechanism matching** (Prompt Engineering Techniques) — the one miss. Now at 38% wrong-rate (3/8 attempts) — the recurring failure mode is defaulting to "add more prose/detail" instead of "replace vague description with concrete examples." Worth drilling specifically before the exam.
- **few-shot-vs-tool-granularity** — still your worst topic by wrong-rate (80%, 4/5 wrong historically) even though you got today's rep right. One correct answer doesn't clear a pattern with this much history — keep it in the weak-spot pool.
- **coordinator-context-passing** — largest volume weak spot (11 attempts, 36% wrong-rate). Both reps today were correct, but the historical rate says this needs more reps before it's solid.

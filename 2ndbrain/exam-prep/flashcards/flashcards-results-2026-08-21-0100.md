# Flashcards results — 2026-08-21

**Scope:** Study-guide note `exam-prep/study-guide-weak-spots/study-guide-weak-spots-2026-08-19-0036/1-2-orchestration-patterns.md` (Multi-Agent Orchestration). 8 new question cards generated from this note and appended to the existing "Agents and workflows" chapter cache (q17-q24), plus 2 reinforcement questions from that chapter's existing weak topics.
**Score:** 10/10

## Q1 — Narrow-decomposition-failure-tracing

**Question:** A multi-agent research system assigned 'renewable energy technologies' decomposes this into only 'solar panel efficiency' and 'wind turbine design.' Each subagent produces thorough, well-sourced research. The final report is comprehensive on solar and wind but missing geothermal, tidal, biomass, and nuclear fusion entirely. What is the root cause?

**Options:**
- The synthesis subagent dropped content during aggregation
- The coordinator's task decomposition was too narrow — it never assigned those subtopics to any subagent (✅ correct, chosen)
- The subagents didn't share memory, so coverage gaps like this are unavoidable
- The web search subagent failed to search broadly enough

**Result:** Correct.

**Explanation:** Coverage gaps trace to the coordinator's decomposition, not the subagents — each subagent researched thoroughly within its assigned scope; the scope itself just never included those subtopics. When output is incomplete in breadth (not depth), that's the coordinator's decomposition, almost always.

## Q2 — Narrow-decomposition-failure-tracing (fix)

**Question:** For the renewable-energy coverage gap (missing geothermal, tidal, biomass, fusion), what is the correct fix?

**Options:**
- Add more subagents to the pipeline so more ground gets covered
- Improve the web search subagent's query formulation
- Better coordinator decomposition covering the full topic breadth — not better search queries, not a stronger synthesis agent, and not more subagents (✅ correct, chosen)
- Add a more capable synthesis subagent that can catch omitted topics on its own

**Result:** Correct.

**Explanation:** This is one of the four explicit exam traps: adding more subagents to fix a decomposition problem doesn't help — new subagents just receive equally narrow assignments from the same flawed decomposition. The fix has to be upstream, in what the coordinator decides to assign in the first place.

## Q3 — Coordinator-responsibilities (dynamic selection)

**Question:** A user asks a hub-and-spoke research system (with search, analysis, and synthesis subagents) a simple factual question. Per the 'dynamic subagent selection' responsibility, what should the coordinator do?

**Options:**
- Always invoke all three subagents for consistency, regardless of query complexity
- Randomly select a subset of subagents to reduce load
- Dynamically select just the subagent(s) actually needed (e.g. only web search) rather than always routing through the full pipeline (✅ correct, chosen)
- Delegate to the synthesis subagent first so it can decide what other subagents are needed

**Result:** Correct.

**Explanation:** Dynamic selection means the coordinator analyzes what a query actually needs — a simple factual question might only need web search, not the full search-analysis-synthesis chain. Always routing through every subagent wastes time and resources on queries that don't need that depth.

## Q4 — Coordinator-responsibilities (scope partitioning)

**Question:** The coordinator delegates research on the same broad topic to two subagents. Per the 'research scope partitioning' responsibility, what should it do?

**Options:**
- Let each subagent decide its own scope independently
- Assign distinct, non-overlapping subtopics or source types to each subagent (e.g. one searches academic papers, another searches news) to minimize duplication (✅ correct, chosen)
- Only use one subagent at a time so partitioning is never needed
- Assign the exact same query to both subagents to cross-validate results

**Result:** Correct.

**Explanation:** Scope partitioning is about avoiding wasted, duplicated effort — each subagent gets a distinct subtopic or source type, so two subagents don't both search the same ground. Assigning the same query to both is the opposite of partitioning; it's redundancy, not cross-validation (this system has no mechanism for reconciling conflicting subagent outputs anyway).

## Q5 — Coordinator-responsibilities (iterative refinement)

**Question:** The coordinator evaluates its synthesis output and finds coverage gaps. Per the exam-tested 'iterative refinement' responsibility, what should happen next?

**Options:**
- The coordinator restarts the entire pipeline from scratch with a brand-new decomposition
- The coordinator accepts the output as final, since the subagents already ran once
- The coordinator re-delegates to search/analysis subagents with targeted queries addressing the gaps, then re-invokes synthesis — repeating until coverage is sufficient (✅ correct, chosen)
- The synthesis subagent requests additional research directly from the search subagent

**Result:** Correct.

**Explanation:** This is an iterative cycle, not a single-shot process — the coordinator evaluates, targets the specific gaps with follow-up queries, and re-invokes synthesis, repeating until coverage is sufficient. Restarting from scratch discards useful partial work, and the synthesis subagent can't request its own follow-up research (only the coordinator assigns tasks — same rule as the direct-communication ban).

## Q6 — Subagent-communication-routing

**Question:** An engineer proposes letting the search subagent pass results directly to the synthesis subagent, skipping a round-trip through the coordinator, to save time. Per the exam's cardinal communication rule, why is this the wrong answer even though it's more efficient?

**Options:**
- Subagents are technically incapable of sending messages to each other in any Claude Code implementation
- It's only a problem once more than two subagents are involved
- Direct subagent-to-subagent communication breaks observability, consistent error handling, and controlled information flow — all communication must route through the coordinator regardless of efficiency gains (✅ correct, chosen)
- It's fine as long as the coordinator is notified afterward

**Result:** Correct.

**Explanation:** The exam treats this as an unbreakable rule regardless of the efficiency argument — even though the real Claude Code implementation technically allows nested parent-child delegation, on the exam any "let subagents talk directly for efficiency" proposal is the wrong answer, because it forfeits the three centralized benefits (observability, consistent error handling, controlled information flow).

## Q7 — Coordinator-context-passing (no shared memory across invocations)

**Question:** The coordinator invokes the web search subagent twice in one session — once for 'solar panel efficiency,' later for 'wind turbine costs.' What does the second invocation know about the first?

**Options:**
- It inherits the coordinator's full conversation history, including the first invocation
- Nothing — every subagent invocation is completely independent; the second call has zero knowledge of the first unless the coordinator explicitly includes it in the new prompt (✅ correct, chosen)
- It can look up the first invocation's results from shared global state
- It automatically retains the first invocation's results in its own memory

**Result:** Correct.

**Explanation:** This is "the single most misunderstood idea in multi-agent systems" per the note: subagents don't share memory between invocations at all, even repeat calls to the same subagent type. No shared global state, no automatic retention — every invocation starts cold unless the coordinator explicitly re-includes what's needed.

## Q8 — Coordinator-context-passing (isolation principle, default access)

**Question:** When the coordinator spawns a subagent, what does that subagent have access to by default?

**Options:**
- The coordinator's system prompt, but not any prior messages
- Only what the coordinator explicitly includes in its prompt — not the coordinator's system prompt, prior conversation, or other subagents' results unless passed explicitly (✅ correct, chosen)
- Results from any other subagent already invoked in the session
- The coordinator's full conversation history automatically

**Result:** Correct.

**Explanation:** This is the isolation principle stated in full: no coordinator system prompt, no prior conversation, no other subagents' results, no shared memory — a subagent's entire world is whatever the coordinator put explicitly into its prompt. This is why the coordinator has to be deliberate: if the synthesis agent needs the search results, they must be passed, not "looked up."

## Q9 — Coordinator-context-passing (existing chapter weak spot, reinforcement)

**Question:** A customer-support coordinator routes a ticket to a billing subagent (which confirms the customer's account tier) and then to a technical subagent (which troubleshoots a feature outage). The technical subagent's response ignores the fact that the customer's tier doesn't include that feature at all. What's the most likely root cause?

**Options:**
- This is expected — subagents aren't supposed to share findings with each other
- The technical subagent's system prompt is missing an instruction to ask about account tier
- The coordinator never passed the billing subagent's account-tier finding into the technical subagent's input (✅ correct, chosen)
- The billing subagent misreported the account tier

**Result:** Correct — breaks a previous 4/6 wrong-rate streak on this exact topic.

**Explanation:** The technical subagent has no way to know about the billing subagent's finding unless the coordinator explicitly forwards it; the fix is never "add an instruction telling the subagent to ask" — subagents can't reach outside their own prompt to gather missing context themselves.

## Q10 — Role-scoped-subagent-splitting (existing chapter weak spot, reinforcement)

**Question:** A code review agent equipped with 18 tools has slow reviews and frequently selects the wrong tool. What is the most effective architectural change?

**Options:**
- Add more detailed descriptions to all 18 tools so the single agent can distinguish them better
- Lower the temperature so tool selection becomes more deterministic
- Split the single agent into role-scoped subagents (e.g. security, style, performance), each with a small tool subset relevant to its role (✅ correct, chosen)
- Reduce to a single generic tool that can perform all 18 functions via parameters

**Result:** Correct.

**Explanation:** When one agent has 18 tools spanning multiple unrelated review dimensions, that's a scope problem (same pattern as the `few-shot-vs-tool-granularity` topic drilled in a previous session), not a description or determinism problem — splitting into role-scoped subagents shrinks each one's tool set to only what's relevant to its job.

## Weakest topics this session

None missed (10/10). `coordinator-context-passing` drops off the top of the aggregate weak-spots list after this session's reinforcement. Remaining confirmed weak spots: `few-shot-vs-tool-granularity` (Prompt Engineering Techniques, 4/4 wrong all-time), `tool-choice/tool-granularity confusion` (67%), `worktree-coordination` (67%), `tool description quality` (50%).

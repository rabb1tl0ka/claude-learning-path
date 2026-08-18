# Study guide — claudecertificationguide mock exam, try 2 (2026-08-18)

Grounded in your own chapter notes. Highest-impact topics first — repeat misses across both attempts get priority over new ones. Each topic marked as grounded in an actual missed question from this import, or in general weak-spot history.

## 1. Coordinator context-passing in multi-agent systems (missed in try 1 *and* try 2 — 3 total misses, 100% wrong-rate)

**Core theory** (from `claude-api/agents-and-workflows/agents-and-workflows.md`): in a hub-and-spoke architecture, subagents don't share memory or communicate directly — the coordinator is the only thing that sees every subagent's output. If a downstream agent is missing information a sibling already produced, the fault is almost always the **coordinator failing to forward it as context**, not the downstream agent's own prompt or the upstream agent's output quality.

**This is the same exact question you missed in try 1** ("A synthesis agent produces a report with several claims that have no source attribution...") — word-for-word identical, and you got it wrong again. This is the strongest signal in either attempt: the concept isn't sticking on repetition alone.

**X vs Y**: "the receiving agent's system prompt needs an instruction" vs. "the coordinator isn't passing the right data forward." You've now defaulted to blaming the receiving/downstream agent three times across two attempts.

**Self-check**: If a synthesis agent drops source attribution even though its inputs were well-sourced, where do you look first — the synthesis agent's prompt, or what the coordinator handed it?

## 2. Tool granularity — recognizing when one tool is doing too many jobs (missed in try 1 *and* try 2 — 2 total misses, 100% wrong-rate)

**Core theory** (from `agents-and-workflows.md`'s "tools should be abstract, not hyper-specialized" combined with `prompt-engineering-techniques.md`'s few-shot guidance): a tool doing three unrelated jobs badly (web scraping, document parsing, code analysis all under one `analyze_content` tool) needs to be **split into purpose-specific tools**, not patched with a better description or more few-shot examples.

**This is also close to word-for-word the same question you missed in try 1.** Both times you picked "improve the tool's description" instead of "split it into purpose-specific tools" — recognizing the fix as *splitting*, not *documenting better*, is the gap.

**Self-check**: a single tool handles web scraping, document parsing, and code analysis, and results are poor across all three. Is the fix better examples/descriptions, or fewer responsibilities per tool?

## 3. Tool description quality as the actual fix for tool-selection failures (2 misses across both attempts, 100% wrong-rate)

**Core theory** (from `claude-api/tool-use-with-claude/introducing-tool-use.md`): the tool-description guidance — 3-4 sentences covering what a tool does, when to use it, what it returns — reads almost identically to how a Skill or Agent description gets written in Claude Code. Same purpose, same failure mode if it's vague: an agent that defaults to a familiar general-purpose tool (Bash) over a purpose-built one (an MCP `query_database` tool) usually isn't broken — its description just isn't giving the model enough reason to prefer it.

**This session's miss**: an MCP server's `query_database` tool for Snowflake gets ignored in favor of raw Bash+CLI calls, even though the MCP tool returns richer structured data. You reached for a system-prompt instruction telling the agent to prefer the tool — a brittle, per-agent patch — instead of fixing the tool's own sparse description, which is the fix that scales to every agent that might connect to that MCP server.

**Self-check**: an agent keeps defaulting to a general-purpose tool over a purpose-built one that does the job better. Do you patch the *agent's* instructions, or the *tool's* description?

## 4. Session-state staleness — when to start fresh vs. keep working in place (new this import)

**Core theory** (from `claude-code-in-action/long-sessions-and-steering.md`): `/compact` and rewind exist because long sessions accumulate context that can go stale (a colleague's changes land after you've already read a file) or bulky (a long side-investigation you don't need anymore). The guidance is to actively manage that staleness — summarize what's still valuable, then re-ground in current state — rather than trusting the existing session to self-correct just by re-reading.

**This session's miss**: 40-minute-old tool results are stale because a colleague pushed changes to those files. You picked "stay in the current session and ask the agent to re-read the changed files" — but the stale results remain in the context window regardless, so the agent can still reference outdated content alongside the fresh re-read. The correct move is a fresh session seeded with a summary of key findings, then re-reading the changed files from that clean state.

**Self-check**: if a long session's context includes both fresh and stale versions of the same file, does re-reading the file fix that, or does the stale copy linger either way?

## 5. Worktree coordination for concurrent edits to a shared file (new this import)

**Core theory** (from `long-sessions-and-steering.md`'s worktrees section): worktrees solve the "two steering wheels, one car" problem by giving each Claude Code session an independent file tree — but a shared file two worktrees both need to modify still needs a sequencing plan, since worktrees isolate the working directory, not the underlying git history each branch will eventually merge back into.

**This session's miss**: two instances both need to modify the same `OrderService.java`. You picked "let both edit independently, resolve the merge conflict later" — allowing two independent, potentially contradictory modifications to accumulate is riskier than coordinating upfront. The correct pattern: one instance completes and merges first, the other rebases onto the updated state before making its own changes — sequencing rather than reconciling after the fact.

**Self-check**: two parallel worktrees both need to touch the same file. Is it safer to let both go independently and merge later, or to sequence who touches it first?

## 6. Hook lifecycle events — knowing which event fires for which moment (new this import)

**Core theory** (from `claude-code-in-action/automating-and-verifying-work.md`'s hooks list): Claude Code fires hooks at specific named lifecycle moments — pre-tool-use, post-tool-use, stop, subagent-stop, and (per that chapter's list) pre-compact/post-compact around compaction specifically. These are fixed, named events, not something you wire up generically against an arbitrary "tool."

**This session's miss**: archiving the full transcript before `/compact` runs needs a **PreCompact** hook — a dedicated event that fires immediately before compaction. You picked "a PreToolUse hook configured on a built-in 'Compact' tool," inventing a tool that doesn't exist rather than recalling the actual dedicated event for this moment.

**Self-check**: you need something to happen right before context gets compacted. Is there a tool being called here to hook `PreToolUse` onto, or is this its own lifecycle moment?

## 7. Writing CLAUDE.md rules that are specific and checkable, not just present (new this import)

**Core theory** (from `long-sessions-and-steering.md`): "effective rules are specific and checkable, not vague — 'follow best practices' tells Claude nothing; a concrete rule with a named alternative does." A rule can exist in CLAUDE.md and still fail in practice if it's not concrete enough to act as an actual check.

**This session's miss**: CLAUDE.md already required integration tests with contract compliance and rollback checks, but Claude sometimes generated unit tests with mocked databases instead. You picked banning mocks outright via a `.claude/rules/` file — over-broad, since mocks are legitimate in plenty of other test contexts. The fix was tightening the *existing* CLAUDE.md rule with concrete criteria (real DB connections, contract assertions) rather than adding a blanket ban elsewhere.

**Self-check**: a CLAUDE.md rule is being followed inconsistently. Is the fix moving it somewhere else, or making the existing rule concrete enough that "did I follow it" has an obvious answer?

---

## Coverage gaps — not in any existing chapter note

These questions don't map to material in this repo's notes. They're real gaps, not weak-spot misses — studying existing notes won't help here.

- **Subagent parallel spawning mechanics** (`fork_session` vs. multiple `Task` tool calls in one coordinator response): "A multi-agent research system has a coordinator that spawns a web search subagent and a document analysis subagent sequentially... How should the architect reduce this latency?" — `fork_session` is for divergent exploration from a shared baseline, not for spawning concurrent subagents; the correct mechanism is emitting multiple `Task` calls in a single response. Not covered by any chapter note.
- **Nullable/optional schema fields as the fix for structured-output fabrication** (missed twice, same underlying concept, different document-extraction scenarios): when a schema marks fields required that aren't always present in the source, the model fabricates plausible values to satisfy the schema. The fix is making those fields nullable/optional, not adding post-extraction validation or splitting into per-document-type schemas. Same domain as the Structured Data Extraction gap flagged in the try-1 study guide — still uncovered.
- **MCP tool consolidation vs. splitting across servers**: "The data platform team has built an MCP server with 22 tools... Agents take 3-4 turns to select the correct tool and frequently choose the wrong transformation. What is the most effective redesign?" — MCP server boundaries are invisible to the agent (all connected tools appear in one flat list), so splitting tools across servers doesn't reduce the agent's real choice set; consolidating near-duplicate tools into one parameterized tool does. Not covered by any chapter note.
- **Claude Code built-in tool selection for codebase-wide search-and-replace** (Grep vs. Glob vs. Bash+sed for finding all references to a renamed API endpoint across 500,000 lines): Grep searches file contents across the whole codebase; Glob only matches file paths/names. Not documented in any chapter note — same gap flagged in the try-1 study guide (Grep/Glob/Edit mechanics), still open.
- **Multi-agent task decomposition as the root cause of incomplete coverage**: "A multi-agent research system produces a report on 'renewable energy technologies' that only covers solar and wind, missing geothermal, tidal, biomass, and nuclear fusion... Where is the root cause?" — the coordinator's own task decomposition (what it assigned subagents to research) is the thing to check first, before assuming a downstream subagent's search or source access failed. Related to, but distinct from, the coordinator-context-passing gap above (topic 1) — that's about *passing* results forward; this is about *assigning* the right scope in the first place.

**2 of this session's misses already have an existing gap-topics note, but weren't seeded into history because `--include-gaps` wasn't passed on this import:**
- Q16 (structured provenance metadata vs. prose footnotes) → matches `exam-prep/gap-topics/structured-data-extraction/structured-claim-source-mapping.md`
- Q49 (`tool_choice` forcing by name vs. `tool_choice: 'any'`) → matches `exam-prep/gap-topics/tool-use-with-claude/tool-choice-forcing.md`

Rerun the import with `--include-gaps` if you want these two folded into the tracked weak-spot signal instead of sitting outside it.

## Suggested study order

1. **Coordinator context-passing** (topic 1) — the single strongest signal in this data: the exact same question, missed twice. This is worth actually drilling until it's automatic, not just re-reading.
2. **Tool granularity** (topic 2) — same pattern: missed twice on essentially the same scenario. Pair with topic 3 (tool description quality) since both live in the same "is this a tool-design problem or a tool-description problem" decision space.
3. **Tool description quality** (topic 3).
4. **Session/worktree state management** (topics 4-5) — new this round, both from `long-sessions-and-steering.md`, good to batch together.
5. **Hook lifecycle events + CLAUDE.md rule specificity** (topics 6-7) — new this round, both from Claude Code configuration material.
6. Then the coverage gaps — the nullable-fields-for-fabrication gap and the Grep/Glob built-in-tools gap are both repeats from try 1's study guide and still open; worth closing those specifically since they've now cost points twice.

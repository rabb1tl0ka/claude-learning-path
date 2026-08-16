## Source
- [Agents and workflows](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287796)
- [Parallelization workflows](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287804)
- [Chaining workflows](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287800)
- [Routing workflows](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287801)
- [Agents and tools](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287803)
- [Environment inspection](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287798)
- [Workflows vs agents](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287794)
- [Quiz on Agents and Workflows](https://anthropic.skilljar.com/claude-with-the-anthropic-api/289130)

## Summary

**Workflows vs. agents — the rule of thumb**
Both are strategies for tasks too big for a single Claude request. Use a **workflow** when you know the precise series of steps needed. Use an **agent** when the steps aren't known ahead of time and Claude needs to plan its own path using a set of tools.

**Workflow patterns** (proven structures other engineers have had success with, not something you have to invent from scratch):
- **Evaluator-optimizer**: a "producer" (e.g. Claude + a CAD library modeling a 3D part from an image) generates output; a "grader" checks it against criteria; if it fails, feedback loops back to the producer until the grader accepts the result.
- **Parallelization**: split one task into independent subtasks that run concurrently (e.g. separately asking Claude to judge suitability of metal/polymer/ceramic/etc. for a part), then join the results in a final aggregator call. Benefits: each subtask prompt stays focused and simple, subtasks are easy to evaluate/improve independently, and the pattern scales well (add/remove subtasks freely).
- **Chaining**: break a big task into a fixed sequence of steps, each a separate Claude call (e.g. search trends → pick topic → research → write script → generate/post video). Also useful as a fix for prompts with many constraints Claude keeps partially ignoring: rather than repeating all constraints in one giant prompt, chain a follow-up call that hands back the previous output and asks for specific, narrow corrections (e.g. "remove emojis, remove AI self-references, adjust tone").
- **Routing**: a first Claude call classifies/categorizes the input (e.g. topic → genre: educational vs. entertainment), then the input is forwarded to a specialized downstream prompt/pipeline suited to that category.

**Agents**: given a task and a set of tools, Claude plans and combines tool calls itself (e.g. combining simple tools like "get current datetime," "add duration," and "set reminder" to answer varied questions, asking the user for more info when needed).
- **Tools should be abstract, not hyper-specialized.** Claude Code itself is the example: it gets generic tools (bash, web fetch, write, read) rather than a "refactor" tool or an "install dependencies" tool, and figures out how to combine the generic tools to accomplish specific goals.
- **Environment inspection matters.** After (and sometimes before) taking an action, Claude needs a way to check the actual resulting state, not just trust whatever the tool call returned — e.g. Computer Use takes a screenshot after every action; Claude Code should read a file before editing it; a video-generation agent might be told to use `whisper.cpp` to check caption timestamps or `ffmpeg` to pull frame screenshots to verify the output actually looks right.

**Workflows vs. agents, compared**: workflows divide big tasks into small, focused steps, giving higher accuracy and much easier testing/evaluation, since the exact steps are known ahead of time. Agents trade that reliability for flexibility — they can handle inputs/requests that weren't anticipated, and can ask the user for more input — but they have a lower success rate and are harder to test. General recommendation: prefer workflows whenever the steps are knowable, and only reach for agents when they're genuinely required, since what most users actually want is something that works reliably, not "a fancy agent."

The session ended with a quiz on these patterns (evaluator-optimizer, parallelization, routing, chaining, workflow vs. agent selection) — Bruno passed 6/7.

## My Insights
- Prefers the term "workflow patterns" over the course's "workflow types" — same idea, different naming instinct.
- While hearing the parallelization example (material suitability judged separately per material), floated an alternative approach out loud — first ask Claude to identify what the part is actually made of, then send a follow-up prompt tailored to that material — before catching himself and realizing he'd misunderstood the original problem (the task is deciding what material to recommend building the part *out of*, not identifying its current material). Self-corrected mid-thought rather than a lasting idea.
- Pushed back on the chaining-workflow framing for fixing constraint-violations: some of those fixes (e.g. stripping emojis) could just be deterministic string processing, no LLM call needed. More pointedly, he finds it notable that this gets presented as a "feature" (prompt chaining) when really it's papering over the fact that the first paid request failed to follow instructions — "just try again" dressed up with a name.
- Flagged environment inspection (giving an agent the ability to check/test its own output) as an important idea worth remembering.
- On the quiz: got one question wrong — a "Claude keeps ignoring some of your rules in a long prompt" scenario — because he initially reached for a routing/categorization fix; the intended answer was to break the task into sequential steps (chaining) instead, since categorization doesn't apply when there's nothing to categorize.

## Ideas
None this session (the parallelization alternative above was raised then explicitly retracted).

## Challenges
- Distinguishing when a "long prompt with many constraints Claude keeps missing" calls for chaining/sequential steps rather than routing — tripped him up on the quiz.

## Actions
- [x] Research how to create custom tools that Claude Code can use (owner: claude) (done same session, see `research/creating-custom-tools-for-claude-code.md`)

## Source
- [Final Assessment](https://anthropic.skilljar.com/claude-with-the-anthropic-api/290899)
- [Course Wrap Up](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287802)

## Summary

This session covers the two closing pieces of the Building with the Claude API course: taking the Final Assessment quiz, and the course's own wrap-up recap.

**Course recap (narration):**
- The course started with an overview of Anthropic's models: Haiku for fast, smaller requests, Sonnet for greater intelligence.
- Covered accessing Claude via the API and the parameters used to steer responses: temperature, stop sequences, and message prefilling.
- Prompt evaluation was emphasized as the single most important practice to carry into real projects: manual spot-checking (running a prompt 10 times and eyeballing it) isn't enough once something's in production, and a simple, Claude-generated eval framework is enough — no need for a heavyweight framework.
- Prompt engineering technique: the top one is simply being clear and direct about what's expected of Claude.
- Tool use was called out as one of the more complex sections, since it dramatically expands what Claude can do.
- Covered two Anthropic-built applications: Claude Code and Computer Use.
- Closed with workflows vs. agents: agents are more exciting but workflows generally give better results and higher accuracy for tasks with a known set of steps.
- Recommended follow-up research areas not covered in the course: agent orchestration, monitoring agent performance, agentic RAG, and evaluation methodologies for both RAG and tools (tool evals work like prompt evals — making sure tool descriptions actually guide Claude the way intended).

## My Insights

- Scored 23/24 on the final assessment, missing only the question on what batch tools do — thought a batch tool was for processing files that exceed normal size limits, when it's actually about accepting multiple tool calls and executing them simultaneously in one request.
- Confirmed preference (previously discussed) for running Claude Code directly in the terminal rather than through an in-editor assistant.

## Ideas
(none this session)

## Challenges
- Batch tools: mixed up "handles large payloads" with the actual definition (concurrent execution of multiple tool calls in a single request).

## Actions
- [ ] Complete the Introduction to Model Context Protocol course (owner: bruno)
- [ ] Complete the Claude Code in Action course (owner: bruno)

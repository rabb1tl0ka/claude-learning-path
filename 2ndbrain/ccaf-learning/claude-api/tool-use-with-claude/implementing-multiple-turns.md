## Source
- [Implementing multiple turns](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287758)

## Summary

Implementing the loop described in the previous lesson as a `run_conversation` function: it takes an initial list of messages, and inside a `while` loop it repeatedly sends them to Claude, checks the response's `stop_reason`, and:
- if it's not `tool_use`, that's the final response — break out and return it to the user
- if it is `tool_use`, run the requested tool, append the resulting `tool_result` block(s) to the message list, and loop again — still inside the same `while` loop

This keeps calling Claude until it stops asking for tools, at which point there's a final answer ready to send back.

## My Insights

(covered under the previous note — same underlying discussion carried across both lessons)

## Ideas

(none new)

## Challenges

(none new — see [multi-turn-conversations-with-tools.md](multi-turn-conversations-with-tools.md))

## Actions

(none new this lesson)

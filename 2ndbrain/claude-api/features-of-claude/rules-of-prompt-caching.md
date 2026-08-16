## Source
- [Rules of prompt caching](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287770)

## Summary

- Caching is not automatic — it requires manually adding a **cache breakpoint** to a block via a `cache_control` field.
- `cache_control` can only be added when a text block is written in **longhand form** (`{"type": "text", "text": "..."}`), not the shorthand string-only form used throughout the rest of the course.
- A breakpoint caches all content up to and including it. A follow-up request only hits the cache if its content up to that same breakpoint is **byte-for-byte identical** — even adding a single word (e.g. "please") invalidates it, forcing a full reprocess.
- Breakpoints can span multiple messages and block types — text, image blocks, tool use, tool results, system prompts, and tool definitions can all carry a `cache_control` field.

## My Insights

- Correctly restated the caching rules back (breakpoints, exact-match requirement, multi-message/multi-type spanning) as a way of checking his own understanding against the video.
- Confirmed for himself that cache key management is entirely handled on Claude's side — the caller never has to think about cache keys/IDs, just where to put the breakpoint.

## Ideas

- None this session.

## Challenges

- Initially unclear on *why* breakpoints aren't automatic / when you'd actually choose to place one — resolved by the video's own explanation as it continued.

## Actions

- None this session.

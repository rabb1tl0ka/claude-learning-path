## Source
- [Temperature](https://anthropic.skilljar.com/claude-with-the-anthropic-api/287728)
- [Course satisfaction survey](https://anthropic.skilljar.com/claude-with-the-anthropic-api/297284)

## Summary
Claude generates text in three steps: tokenize the input, predict probabilities for each possible next token, then sample a token based on those probabilities (repeated to build the full response).

**Temperature** is a decimal between 0 and 1 that reshapes that probability distribution before sampling:
- **0** → deterministic: always pick the highest-probability token.
- **Higher values** → flatten the distribution, increasing the odds of lower-probability tokens getting picked (more variety/creativity).

**Suggested ranges by task:**
- **0-30%**: factual responses, coding assistance, data extraction — certainty matters, no creativity needed.
- **Medium**: summarization, educational content.
- **80-100%**: brainstorming, creative writing, jokes — where unexpected word choices are the point.

**Implementation**: add a `temperature` argument to the `chat` function, defaulting to `1.0` (favors creativity), passed through into the `params` object of the API call.

Also did the course satisfaction survey this session: rated satisfaction and likelihood-to-recommend positively, with feedback requesting more real-code usage in the course.

## My Insights
Initially confused about *why* temperature was being explained via the input side rather than the output — expected it to describe how the model's own generated response gets sampled, not how it interprets what I typed. Resolved once the video connected temperature back to the sampling step of Claude's own text generation (input tokenization/prediction is just necessary background, not what temperature acts on).

Confirmed Claude Code itself doesn't let you set a temperature parameter directly — tested this live during the video.

## Ideas

## Challenges

## Actions

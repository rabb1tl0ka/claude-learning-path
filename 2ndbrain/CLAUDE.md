# 2ndbrain — study notes vault

Notes for the Claude Partner Network learning path. See the repo-root `CLAUDE.md`
for the overall project structure and course list.

## Structure

- One directory per course, kebab-case, matching the names in the root CLAUDE.md
  course table (e.g. `claude-api`, `model-context-protocol`).
- One markdown note per chapter inside that course's directory, kebab-case
  filename (e.g. `what-are-skills.md`).

## Per-chapter workflow

For each chapter, Bruno will give:
1. The URL for the course chapter
2. Claude summarizes the chapter content
3. Bruno adds his own notes/insights

Source material for the summary comes from Gemini transcripts of Bruno's
self-scheduled Google Meet sessions (screen-shared tab of the course video,
Gemini transcription on) — the video's narration shows up in the transcript as
"Bruno Coelho's Presentation: ...".

## Chapter note structure

Each chapter note is a single `.md` file with these sections, in order:

```markdown
## Source
<optional: one markdown link per matching lesson on the course platform, e.g.
`- [Temperature](https://anthropic.skilljar.com/...)`. Only present when a
`.course-toc.json` lookup found a match — see the `learning-recap` skill.>

## Summary
<summary of the chapter material, with the parts Bruno flagged as important highlighted>

## My Insights
<Bruno's own take, connections to other things he knows, disagreements, etc.>

## Ideas
- Things Bruno wants to try/build, sparked by this chapter (not necessarily actionable yet)

## Challenges
- Things Bruno found hard or wants to dig into further

## Actions
- [ ] Concrete thing Bruno needs to do (owner: bruno) (due: yyyy-MM-dd)
- [ ] Same, without a due date if none applies
- [ ] Something Bruno asked Claude to go do (owner: claude)
```

The `## Actions` section follows the generic convention read by the
`learning-actions` skill (`~/.claude/skills/learning-actions/SKILL.md`) —
`- [ ]`/`- [x]` checkboxes, optional `(owner: name)` and `(due: yyyy-MM-dd)`
suffixes. `(owner: ...)` defaults to `bruno` when omitted — older chapter notes
written before this tag existed don't need to be retrofitted. During Gemini
session recording, Bruno sometimes narrates asides that are direct requests to
Claude ("Claude, go research X and save it as a note") rather than his own
commitments — those get `(owner: claude)` and are handled per the
`learning-recap` skill's rules (may be executed immediately during that same
recap run, rather than staying open). Running `/learning-actions` from the
repo root gives a consolidated view of every open action across all chapters.

## Progress

**Current course:** Building with the Claude API (`claude-api`)

**Chapters completed:**
- [x] What are Skills — `claude-code-agent-skills/what-are-skills.md`
- [x] Output styles: custom persona — `claude-code-agent-skills/output-styles-custom-persona.md`
- [x] System prompt vs CLAUDE.md vs output styles — `claude-code-agent-skills/system-prompt-vs-claude-md-vs-output-styles.md`
- [x] CLAUDE.md load order — `claude-code-agent-skills/claude-md-load-order.md`
- [x] Output style per directory — `claude-code-agent-skills/output-style-per-directory.md`
- [x] Temperature — `claude-api/temperature.md`
- [x] Response Streaming — `claude-api/response-streaming.md`
- [x] Structured Data — `claude-api/structured-data.md`
- [x] Prompt Evaluation — `claude-api/prompt-evaluation.md`
- [x] Prompt Engineering Techniques — `claude-api/prompt-engineering-techniques.md`
- [x] Introducing Tool Use — `claude-api/introducing-tool-use.md`

Update this section whenever a new chapter note is completed: mark it `[x]`, add
the file link, and update "Current course" when moving to the next course.

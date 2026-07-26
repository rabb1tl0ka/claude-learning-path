# Claude Partner Network Learning Path

Bruno's study repo for Anthropic's [Claude Partner Network learning path](https://anthropic.skilljar.com/page/claude-partner-network-learning-path) — the introductory foundation toward Claude Partner Network certification.

**Goal:** complete all 4 courses, then sit the certification exam.

## Courses

| # | Course | Status | Course dir name |
|---|--------|--------|------------------|
| 1 | Introduction to agent skills | Complete | `claude-code-agent-skills` |
| 2 | Building with the Claude API | In progress | `claude-api` |
| 3 | Introduction to Model Context Protocol | Not started | `model-context-protocol` |
| 4 | Claude Code in Action | Not started | `claude-code-in-action` |

## Structure

- `2ndbrain/` — study notes vault. One directory per course, one section subfolder per course-TOC section (where applicable), one `.md` file per chapter. Kept separate so it stays a portable, code-free knowledge base. See `2ndbrain/CLAUDE.md` for the note-taking workflow.
- `code/` — code written while working through course chapters. Mirrors `2ndbrain/`'s course dirs 1:1, but only populated where a chapter actually involves writing code.

Directory naming is kebab-case throughout — no exceptions (e.g. `claude-api`, `what-are-skills.md`, never `Claude API`).

## How notes get made

Study sessions are self-recorded Google Meets (course video screen-shared, Gemini transcription on) with live narration and commentary. Claude turns each transcript into a chapter note — Summary, My Insights, Ideas, Challenges, Actions — via the `learning-recap` skill. See `2ndbrain/CLAUDE.md` for the full chapter note format and `CLAUDE.md` for repo conventions this is built on.

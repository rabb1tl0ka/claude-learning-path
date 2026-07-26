# Claude Partner Network Learning Path

Bruno's study repo for Anthropic's Claude Partner Network learning path
(https://anthropic.skilljar.com/page/claude-partner-network-learning-path) — the
introductory foundation toward Claude Partner Network certification.

## Goal

Complete all 4 courses, then sit the certification exam.

## Courses

| # | Course | Status | Course dir name |
|---|--------|--------|------------------|
| 1 | Introduction to agent skills | Complete | `claude-code-agent-skills` |
| 2 | Building with the Claude API | In progress | `claude-api` |
| 3 | Introduction to Model Context Protocol | Not started | `model-context-protocol` |
| 4 | Claude Code in Action | Not started | `claude-code-in-action` |

## Directory naming convention

**All lowercase, dashes instead of spaces (kebab-case). No exceptions.**

This applies to every directory under this repo: course dirs, chapter dirs, everything.
e.g. `claude-api`, `what-are-skills.md` — never `Claude API` or `What Are Skills`.

## Top-level structure

- `2ndbrain/` — study notes vault, one dir per course, one `.md` file per chapter.
  Kept separate so it stays a portable, code-free knowledge base. See
  `2ndbrain/CLAUDE.md` for the note-taking workflow.
- `code/` — code developed while working through course chapters. Mirrors the
  course dir names in `2ndbrain/` 1:1 (e.g. `code/claude-api/` pairs with
  `2ndbrain/claude-api/`), but only gets populated for chapters that actually
  involve writing code — don't pre-create empty chapter dirs.

## Progress

**Current course:** Building with the Claude API (`claude-api`)

Update this table and the "Current course" line whenever a course status changes.

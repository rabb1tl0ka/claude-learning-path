# Claude Partner Network Learning Path

Bruno's study repo for Anthropic's Claude Partner Network learning path
(https://anthropic.skilljar.com/page/claude-partner-network-learning-path) — the
introductory foundation toward Claude Partner Network certification.

## Goal

Complete all 4 courses, then prepare with practice exams, then sit the certification exam.

## Courses

All course dirs live under `2ndbrain/ccaf-learning/`.

| # | Course | Status | Course dir name |
|---|--------|--------|------------------|
| 1 | Introduction to agent skills | Complete | `claude-code-agent-skills` |
| 2 | Building with the Claude API | Complete | `claude-api` |
| 3 | Introduction to Model Context Protocol | Complete | `model-context-protocol` |
| 4 | Claude Code in Action | Complete | `claude-code-in-action` |

## Directory naming convention

**All lowercase, dashes instead of spaces (kebab-case). No exceptions.**

This applies to every directory under this repo: course dirs, chapter dirs, everything.
e.g. `claude-api`, `what-are-skills.md` — never `Claude API` or `What Are Skills`.

## Top-level structure

- `2ndbrain/` — study notes vault, one dir per course, one `.md` file per chapter.
  Kept separate so it stays a portable, code-free knowledge base. See
  `2ndbrain/CLAUDE.md` for the note-taking workflow.
  - `2ndbrain/ccaf-learning/` — the course dirs themselves (e.g. `claude-api/`),
    plus first-class topic dirs for CCAF-relevant material the courses don't
    cover (e.g. `structured-data-extraction/`) — the actual exam (CCAF) is
    scoped broader than the 4 official courses, so both live at the same tier.
    See `2ndbrain/CLAUDE.md` for the full structure.
- `code/` — code developed while working through course chapters. Mirrors the
  course dir names in `2ndbrain/ccaf-learning/` 1:1 (e.g. `code/claude-api/`
  pairs with `2ndbrain/ccaf-learning/claude-api/`), but only gets populated
  for chapters that actually involve writing code — don't pre-create empty
  chapter dirs.
- `2ndbrain/exam-prep/` — practice-exam attempts and mistake reviews ahead of
  the actual certification exam. All 4 courses are done, so this is the
  current phase of the repo. Lives inside `2ndbrain/` (not a sibling of it)
  specifically so exam prep can draw on the accumulated notes and research in
  `2ndbrain/ccaf-learning/`, rather than starting from a blank slate — see
  `2ndbrain/CLAUDE.md` for how it's organized.

## Progress

**Current phase:** Exam prep (`2ndbrain/exam-prep/`) — all 4 courses complete, working through practice tests before sitting the certification exam.

Update this table and the "Current phase" line whenever a course status changes or the exam is sat.

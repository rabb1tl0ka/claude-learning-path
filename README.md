# Claude Partner Network Learning Path

Bruno's study repo for Anthropic's [Claude Partner Network learning path](https://anthropic.skilljar.com/page/claude-partner-network-learning-path).

**Goal:** get certified as a **Claude Certified Architect – Foundations (CCA-F)**. The 4 Claude Partner Network courses are the on-ramp, not the finish line — the real exam covers Claude Code, the Claude Agent SDK, the Claude API, and MCP more broadly than those courses teach, so this repo also tracks self-directed research into whatever gaps the exam turns up.

## Where things stand

All 4 courses are **complete**. The repo is now in its **exam-prep phase**: taking mock exams, analyzing what's actually going wrong, closing gaps with targeted notes, and drilling flashcards until it sticks. See `2ndbrain/exam-prep/README.md` for the ranked list of practice exams and the full study loop, and the root `CLAUDE.md` course table for per-course status.

## How to explore this repo

- **`2ndbrain/`** — the study notes vault. Start here to read what's been learned.
  - `2ndbrain/ccaf-learning/` — one directory per course (`claude-api/`, `model-context-protocol/`, `claude-code-agent-skills/`, `claude-code-in-action/`), each with one `.md` chapter note per lesson. Also holds **gap-topic directories** at the same tier (e.g. `structured-data-extraction/`, `tool-choice-forcing/`) — self-directed research on CCA-F material the courses never covered.
  - `2ndbrain/exam-prep/` — mock-exam attempts, failure-pattern analyses, and the flashcards system (`.flashcards/`: cache, weak-spot tracking, generated study guides).
  - See `2ndbrain/CLAUDE.md` for the exact chapter-note format and directory conventions.
- **`code/`** — code written while working through chapters, mirroring `2ndbrain/ccaf-learning/`'s course dirs 1:1. Only populated for chapters that actually involved writing code.
- **`CLAUDE.md`** (root) and **`2ndbrain/CLAUDE.md`** — the operative instructions Claude follows in this repo: naming conventions, note structure, workflows. Read these if you want the ground truth on how the repo is organized, rather than relying on this README staying in sync.

Directory naming is kebab-case throughout, no exceptions.

## Skills available in this repo

These are Claude Code skills scoped to this project (`.claude/skills/`) that drive the study workflow:

| Skill | What it does |
|---|---|
| `/learning-recap` | Turns a Gemini transcript of a self-recorded "watch the course + narrate" session into a chapter note (Summary / My Insights / Ideas / Challenges / Actions). |
| `/flashcards` | Quizzes you with multiple-choice flashcards generated from the chapter and gap-topic notes; tracks weak spots over time. Also imports missed questions from a real mock-exam file to seed weak spots and generate a targeted study guide (`/flashcards import <path>`). |
| `/exam-analysis` | Reads a mock-exam result file and reports *why* you're missing questions (recurring reasoning mistakes, not just which domain scored low) plus what you consistently get right. Flags real coverage gaps — questions that don't map to any existing note. |

Global skills (available in any repo, not specific to this one) that also come up here include `/action-board` (consolidated view of open `## Actions` items across all chapter notes).

## The study loop, in short

1. Take a mock exam → `/exam-analysis <result-file>` to see why you missed what you missed.
2. `/flashcards import <result-file>` to seed weak spots and get a grounded study guide.
3. Read the study guide, research any flagged coverage gaps into `2ndbrain/ccaf-learning/`.
4. `/flashcards` to quiz yourself on the topics just studied.
5. Repeat with the next mock exam until the weak-spots list flattens out, then book the real exam.

Full detail (ranked practice-exam list, exam format from someone who's passed) lives in `2ndbrain/exam-prep/README.md`.

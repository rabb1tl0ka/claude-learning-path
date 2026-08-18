# Exam prep — instructions for Claude

See `README.md` in this directory for the ranked practice-exam list, the full study loop, and the real exam's format. This file is the operative version — what Claude should actually do, not what Bruno reads.

## What to do here

- When Bruno shares mock-exam results (pasted text, a file path, a PDF) — anywhere in conversation, not just when explicitly asked — offer to run `/flashcards import <path>` from `2ndbrain/` rather than parsing the results ad hoc. That's the mechanism that seeds `.flashcards/history.jsonl`, regenerates weak-spots, and writes a study guide grounded in the actual chapter notes.
- Raw exam-result files Bruno hands over for import go in `2ndbrain/exam-prep/attempts/<source-slug>-<date>.<ext>` (e.g. `attempts/cyberskill-2026-08-20.pdf`) — keep them, don't delete after import, so there's a paper trail of what was fed into `/flashcards import` and when.
- `.flashcards/` itself (cache, history, weak-spots, study guides) lives at `2ndbrain/.flashcards/`, not inside this directory — this dir only holds the raw attempt files, the `flashcards/` results archive below, and this README/CLAUDE.md pair.
- `flashcards/flashcards-results-<date-time>.md` — one file per `/flashcards` quiz session. This repo routes results here (instead of the skill's default `.flashcards/results/`) via `2ndbrain/.flashcards-config.json`'s `resultsDir` key, so it's a visible, human-facing archive rather than cache — the skill itself has no hardcoded knowledge of this repo's directory names. Each file compiles every question asked that session with its full teaching explanation (concept, why the right answer's right, why a wrong pick's reasoning failed) — not just a pass/fail log. `.flashcards/weak-spots.md` still owns the aggregate wrong-rate-by-topic table; this is the per-session teaching record.
- If Bruno mentions a new practice exam or resource surfacing in `#claude-learning-path` (Slack `C0B6AP4ULGK`), update the ranked list in `README.md` rather than just answering in chat and letting it evaporate.
- This directory has no per-course chapter-note structure (no TOC, no sections) — it's exam attempts and prep tracking, not lesson notes, so none of `2ndbrain/CLAUDE.md`'s chapter-note conventions apply here. Coverage-gap research notes (see below) aren't an exception to this anymore — they live in `2ndbrain/ccaf-learning/`, not in this directory.

## Coverage gaps

The actual certification is the **Claude Certified Architect – Foundations (CCAF)** exam — scoped to Claude Code, the Claude Agent SDK, the Claude API, and MCP broadly, not just the 4 official Claude Learning Path courses. So mock exams testing material beyond those 4 courses are testing real exam scope, not noise. `/exam-analysis` and `/flashcards import` both flag questions that don't map to any existing chapter note as **coverage gaps** rather than forcing a bad match — these are real content the courses never taught, not weak spots in material Bruno already studied, but they're still material worth knowing cold for the exam.

When a coverage gap is identified and Bruno wants to research it, the note goes in `2ndbrain/ccaf-learning/`, at the same tier as the course dirs (e.g. `ccaf-learning/structured-data-extraction/`, `ccaf-learning/claude-code-built-in-tools/`) — not nested under `exam-prep/`. This used to be `exam-prep/gap-topics/<topic>/`; it was promoted once the CCAF scope was confirmed, since gap topics are real exam material, not a separate/optional bucket. See `2ndbrain/CLAUDE.md`'s Structure section for the full convention:
- One subdirectory per gap topic under `ccaf-learning/`, kebab-case.
- One `.md` file per distinct concept within that topic, kebab-case filename (e.g. `structured-data-extraction/confidence-calibration.md`), same granularity as a chapter note — don't lump an entire topic into one giant file.
- Each note follows the same section structure as a course chapter note (`## Summary`, `## My Insights`, `## Ideas`, `## Challenges`, `## Actions` — see `2ndbrain/CLAUDE.md`), **except no `## Source` section**, since there's no course lesson to link — this is self-directed research, not a course chapter being summarized.
- Don't pre-create empty topic directories or files before there's actual research to put in them — same "don't pre-create" rule as chapter dirs.

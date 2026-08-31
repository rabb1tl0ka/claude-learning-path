# Exam prep — CCA-F certification

All 4 Claude Partner Network Learning Path courses are done. This directory tracks the last stretch: practice exams, mistakes, and study guides before sitting the real **Claude Certified Architect – Foundations (CCA-F)** exam.

## Practice exams, ranked

Per [João Correia](https://lokahq.slack.com) in `#claude-learning-path` (`C0B6AP4ULGK`) — he passed the real exam and compared all of these directly against it:

| Rank | Exam                                                                                                                                                              | Access                                   | Verdict                                                                                                                                                                      |
| ---- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | https://practice.cyberskill.world/practice/ccaf                                                                                                                   | Free, open web                           | **Closest to the real exam** in both style and difficulty — "if you can pass this one comfortably, you're ready." Multiple later passers independently rated it highest too. |
| 2    | [claudecertificationguide.com/mock-exam](https://claudecertificationguide.com/mock-exam)                                                                          | Free, open web                           | Very solid                                                                                                                                                                   |
| 3    | [exampro.co/cca-f](https://www.exampro.co/cca-f)                                                                                                                  | Free, open web                           | Very solid                                                                                                                                                                   |
| 4    | [certsafari.com/anthropic/claude-certified-architect](https://www.certsafari.com/anthropic/claude-certified-architect)                                            | Free, open web                           | Solid, a bit less so                                                                                                                                                         |
| 5    | [Udemy: "Anthropic Claude Certified Architect – 3 Full Practice Exams"](https://www.udemy.com/course/anthropic-claude-certified-architect-3-full-practice-exams/) | Needs a Loka-forms Udemy license request | Weakest — heavier language, answer sometimes guessable from phrasing                                                                                                         |

**Start with CyberSkill.** Full detail on these, plus other resources (official sample-questions PDF, a centralized Notion page, a NotebookLM study aid) is saved in memory — ask Claude to recall the exam-prep resources if this table ever needs refreshing.

## The study loop

1. **Take a mock exam** (CyberSkill first).
2. **Capture your results** — export/screenshot/copy the questions you got wrong, plus your answer and the correct one if shown. Text file or PDF both work.
3. **Analyze failure patterns**: run `/exam-analysis <path-to-file>`. This reports *why* you're missing questions (recurring reasoning mistakes, not just which domain scored low), what you consistently get right, and saves a dated analysis note next to the source file.
4. **Import the misses**: from inside `2ndbrain/`, run `/flashcards import <path-to-file>`. This:
   - Matches each missed question to the relevant chapter or topic note under `ccaf-learning/` (course dirs like `claude-api/`, `model-context-protocol/`, `claude-code-in-action/`, plus first-class gap-topic dirs like `structured-data-extraction/`)
   - Seeds `.flashcards/history.jsonl` with those real misses (tagged so they're distinguishable from self-generated quiz attempts)
   - Regenerates `.flashcards/weak-spots.md`
   - Writes a dated study guide to `.flashcards/study-guides/` — grounded in your own notes, weakest topics first, with a suggested study order and self-check questions
   - Flags any question that doesn't map to an existing note at all (a real coverage gap, not just a weak spot)
5. **Read the study guide**, refresh the weak topics.
6. **Quiz yourself**: run `/flashcards` scoped to just the topics touched by the import, to test whether it actually sank in.
7. **Repeat** with the next mock exam (2nd choice: claudecertificationguide.com, then exampro.co, then certsafari.com). Each import call builds on the same `history.jsonl`, so weak-spot weighting keeps compounding across exams rather than resetting each time.
8. Once you're consistently passing CyberSkill comfortably and the weak-spots list has flattened out, book the real exam.

## Exam format (from someone who's passed)

- 6 possible scenarios: Customer Support Resolution Agent, Multi-Agent Research System, Structured Data Extraction, Code Generation with Claude Code, Claude Code for Continuous Integration, Developer Productivity with Claude.
- You get 4 of the 6 at random, 15 multiple-choice questions per scenario, grouped in sequence (read the scenario context once, applies to all 15 questions).
- Questions test judgment more than rote recall — when/how to escalate to a human, keeping an agentic workflow on-rails, avoiding context dilution, that kind of thing.
- Real exam difficulty tracks closely with these practice exams.

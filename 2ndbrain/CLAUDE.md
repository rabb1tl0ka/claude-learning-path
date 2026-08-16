# 2ndbrain — study notes vault

Notes for the Claude Partner Network learning path. See the repo-root `CLAUDE.md`
for the overall project structure and course list.

## Structure

- One directory per course, kebab-case, matching the names in the root CLAUDE.md
  course table (e.g. `claude-api`, `model-context-protocol`).
- If the course's `.course-toc.json` tags entries with a `section` field (see
  **Course TOC lookup** in the `learning-recap` skill), one subdirectory per
  section inside the course directory, kebab-case, named after that section
  (e.g. `claude-api/tool-use-with-claude/`). Chapter notes live inside their
  section directory, kebab-case filename (e.g.
  `tool-use-with-claude/introducing-tool-use.md`).
- If the course has no TOC file, or the TOC has no `section` field, chapter
  notes live directly in the course directory (flat), kebab-case filename
  (e.g. `claude-code-agent-skills/what-are-skills.md`).
- Standalone ad-hoc research notes (see **Handling `owner: claude` actions**
  in the `learning-recap` skill) always live in a `research/` subdirectory,
  never loose alongside chapter notes: `<section-dir>/research/<note>.md` when
  a section applies (e.g. `tool-use-with-claude/research/tool-loading-strategy.md`),
  or `<course-dir>/research/<note>.md` at the course root when the note isn't
  tied to one specific lesson (no section to nest it under). This keeps each
  section/course dir's top level to just its chapter notes.

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

**Current course:** Claude Code in Action (`claude-code-in-action`) — Complete. All 4 courses are now done; the repo is moving into an exam-prep phase, tracked in `exam-prep/` (a child of `2ndbrain/`, not a course dir — see root `CLAUDE.md`). It lives here rather than as a sibling of `2ndbrain/` specifically so exam prep can pull on the chapter notes and research already built up across the other course dirs.

**Building with the Claude API — Complete** (all sections finished, see chapter list below).

**Introduction to Model Context Protocol — Complete.** A standalone course on the
Claude Partner Network Learning Path (skilljar id `303756`), distinct from the
`claude-api` course. Note: `claude-api/model-context-protocol/mcp-overview.md`
covers MCP as a *section within the Building with the Claude API course* and is
unrelated to this course. This course was consumed in a single session (Bruno
found it duplicated the `claude-api` MCP section and blew through it, including
its final assessment) — see
`model-context-protocol/course-overview-and-assessment.md` for the one whole-course
chapter note.

**Chapters completed:**
- [x] What are Skills — `claude-code-agent-skills/what-are-skills.md`
- [x] Output styles: custom persona — `claude-code-agent-skills/output-styles-custom-persona.md`
- [x] System prompt vs CLAUDE.md vs output styles — `claude-code-agent-skills/system-prompt-vs-claude-md-vs-output-styles.md`
- [x] CLAUDE.md load order — `claude-code-agent-skills/claude-md-load-order.md`
- [x] Output style per directory — `claude-code-agent-skills/output-style-per-directory.md`
- [x] Accessing the API — no chapter note (covered in an untracked pre-repo session, no transcript available)
- [x] Getting an API key — no chapter note (covered in an untracked pre-repo session, no transcript available)
- [x] Making a request — no chapter note (covered in an untracked pre-repo session, no transcript available)
- [x] Multi-Turn conversations — no chapter note (covered in an untracked pre-repo session, no transcript available)
- [x] Chat exercise — no chapter note (covered in an untracked pre-repo session, no transcript available)
- [x] System prompts — `claude-api/accessing-claude-with-the-api/system-prompt-api-vs-claude-code-sdk.md` (ad-hoc note only, not a full chapter note)
- [x] System prompts exercise — no chapter note (covered in an untracked pre-repo session, no transcript available)
- [x] Temperature — `claude-api/accessing-claude-with-the-api/temperature.md`
- [x] Response Streaming — `claude-api/accessing-claude-with-the-api/response-streaming.md`
- [x] Structured Data — `claude-api/accessing-claude-with-the-api/structured-data.md`
- [x] Prompt Evaluation — `claude-api/prompt-evaluation/prompt-evaluation.md`
- [x] Exercise on prompt evals — no chapter note
- [x] Prompt Engineering Techniques — `claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md`
- [x] Introducing Tool Use — `claude-api/tool-use-with-claude/introducing-tool-use.md`
- [x] Multi-turn conversations with tools — `claude-api/tool-use-with-claude/multi-turn-conversations-with-tools.md`
- [x] Implementing multiple turns — `claude-api/tool-use-with-claude/implementing-multiple-turns.md`
- [x] Using multiple tools — `claude-api/tool-use-with-claude/using-multiple-tools.md`
- [x] Fine grained tool calling — `claude-api/tool-use-with-claude/fine-grained-tool-calling.md`
- [x] The text edit tool — `claude-api/tool-use-with-claude/text-editor-tool.md`
- [x] The web search tool — `claude-api/tool-use-with-claude/web-search-tool.md`
- [x] Quiz on tool use with Claude — `claude-api/tool-use-with-claude/quiz-on-tool-use-with-claude.md`
- [x] Introducing Retrieval Augmented Generation — `claude-api/rag-and-agentic-search/introducing-retrieval-augmented-generation.md`
- [x] Text chunking strategies — `claude-api/rag-and-agentic-search/text-chunking-strategies.md`
- [x] Text embeddings — `claude-api/rag-and-agentic-search/text-embeddings.md`
- [x] The full RAG flow — `claude-api/rag-and-agentic-search/the-full-rag-flow.md`
- [x] Implementing the RAG flow — `claude-api/rag-and-agentic-search/implementing-the-rag-flow.md`
- [x] BM25 lexical search — `claude-api/rag-and-agentic-search/bm25-lexical-search.md`
- [x] A Multi-Index RAG pipeline — `claude-api/rag-and-agentic-search/a-multi-index-rag-pipeline.md`
- [x] Extended thinking — `claude-api/features-of-claude/extended-thinking.md`
- [x] Image support — `claude-api/features-of-claude/image-support.md`
- [x] PDF support — `claude-api/features-of-claude/pdf-support.md`
- [x] Citations — `claude-api/features-of-claude/citations.md`
- [x] Prompt caching — `claude-api/features-of-claude/prompt-caching.md`
- [x] Rules of prompt caching — `claude-api/features-of-claude/rules-of-prompt-caching.md`
- [x] Prompt caching in action — `claude-api/features-of-claude/prompt-caching-in-action.md`
- [x] Code execution and the Files API — `claude-api/features-of-claude/code-execution-and-the-files-api.md`
- [x] Quiz on features of Claude — `claude-api/features-of-claude/quiz-on-features-of-claude.md`
- [x] Introducing MCP — `claude-api/model-context-protocol/mcp-overview.md`
- [x] MCP clients — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Project setup — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Defining tools with MCP — `claude-api/model-context-protocol/mcp-overview.md`
- [x] The server inspector — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Implementing a client — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Defining resources — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Accessing resources — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Defining prompts — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Prompts in the client — `claude-api/model-context-protocol/mcp-overview.md`
- [x] MCP review — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Quiz on Model Context Protocol — `claude-api/model-context-protocol/mcp-overview.md`
- [x] Anthropic apps — `claude-api/anthropic-apps-claude-code-and-computer-use/anthropic-apps.md`
- [x] Agents and workflows (covers Agents and workflows, Parallelization workflows, Chaining workflows, Routing workflows, Agents and tools, Environment inspection, Workflows vs agents, Quiz on Agents and Workflows) — `claude-api/agents-and-workflows/agents-and-workflows.md`
- [x] Final Assessment and Course Wrap Up — `claude-api/final-assessment-and-wrap-up.md` (last chapter of the `claude-api` course — course marked Complete)
- [x] Long-running sessions, steering, and CLAUDE.md configuration (covers long session management/scoping, rewind, goal, loop, worktrees, CLAUDE.md configuration) — `claude-code-in-action/long-sessions-and-steering.md`
- [x] Automating and verifying work (covers verification skills, permission modes, auto mode, hooks, verifying unsupervised runs) — `claude-code-in-action/automating-and-verifying-work.md`
- [x] Sharing and scaling Claude Code (covers routines, headless mode, agent SDK, automated PR reviews, plugins, and the course's final assessment) — `claude-code-in-action/sharing-and-scaling-claude-code.md`

Update this section whenever a new chapter note is completed: mark it `[x]`, add
the file link, and update "Current course" when moving to the next course.

# Flashcards results — 2026-08-18 19:00

**Score: 8/10**

Unscoped quiz weighted toward confirmed weak topics from quiz history (root-cause-to-mechanism matching, tool description quality, tool-choice/tool-granularity confusion, effective-claude-md-rules), plus a few never-attempted topics sampled for breadth (workflows vs. agents rule of thumb, hook exit codes, claude-md-hierarchy).

## 1. Tool description quality — Building with the Claude API / Introducing Tool Use

**Q:** Claude is repeatedly calling the wrong tool out of two similarly-named tools in a set. Per the course's guidance on tool descriptions, what's the most direct fix?

**Options:** Add second-call check / Lower temperature / Rewrite descriptions / Add few-shot examples

**Your answer:** Rewrite descriptions — **Correct**

The fix for tool-selection confusion is almost always the tool's own description, not a compensating layer bolted on afterward. A good description covers what the tool does, when to use it, and what it returns — enough that two similarly-named tools become distinguishable on the spec alone. Adding a second verification call or more few-shot examples treats the symptom (wrong pick) without touching the cause (ambiguous spec), and lowering temperature just makes the same ambiguous choice more consistently wrong.

## 2. Tool-choice/tool-granularity confusion — Building with the Claude API / Introducing Tool Use

**Q:** Two tools overlap in what they can do (e.g. one sets a reminder for 'a date' and another for 'a duration from now'), and Claude picks the wrong one under time pressure. What's actually wrong here?

**Options:** Add few-shot example / Add verification pass / Longer descriptions / Overlapping granularity

**Your answer:** Longer descriptions — **Wrong** (correct: Overlapping granularity)

The problem isn't that the descriptions are too short or vague — it's that the two tools' jobs overlap. "Set a reminder for a date" and "set a reminder for a duration from now" aren't cleanly separated tasks; there's a real ambiguity zone between them regardless of how well-written the description is. No amount of extra description text resolves genuine granularity overlap — the fix is to redesign the tools so each owns a distinct, non-overlapping responsibility.

This is the same reflexive pattern flagged in weak-spots history (`tool-choice/tool-granularity confusion`, `few-shot-vs-tool-granularity`): defaulting to "patch the documentation/examples" when the actual fix is "split/redesign the tool boundaries." **Description problem** = the tool's job is clear but poorly explained; **granularity problem** = the tool's job itself isn't cleanly scoped.

## 3. Multi-block assistant messages — Building with the Claude API / Introducing Tool Use

**Q:** When Claude decides to use a tool, what does the assistant message actually contain?

**Options:** Single plain-text string / Only tool_use block / A tool_result block / Multiple content blocks

**Your answer:** Multiple content blocks — **Correct**

An assistant message on a tool-call turn is a list of content blocks, not a single thing. Claude commonly emits a `text` block (e.g. reasoning or a brief note to the user) followed by a `tool_use` block carrying the tool's `id`, `name`, and `input`. That's why developer code has to iterate over `content` rather than assume one shape — and why message-history code that only grabs "the text" silently drops the tool_use block, breaking the next turn.

## 4. Effective CLAUDE.md rules — Claude Code in Action / Long-running sessions, steering, and CLAUDE.md configuration

**Q:** Which CLAUDE.md rule is written effectively, per the guidance that rules should be specific and checkable?

**Options:** Be careful / No default exports / Specific route path / Follow best practices

**Your answer:** Specific route path — **Correct**

"Put new API routes in `src/api/handlers`, one per file" is specific and checkable — compliance can be verified by looking at the file location and count. "Be careful," "follow best practices," and "don't use X" (with no alternative) are all vague enough that compliance is unfalsifiable.

## 5. Root-cause-to-mechanism matching (1/3) — Building with the Claude API / Prompt Engineering Techniques

**Q:** A rubric criterion is written as vague prose (e.g. 'flag severe issues') and Claude's judgments are inconsistent across similar cases. What's the correct fix?

**Options:** Add concrete examples / Add a second pass / Lower temperature / More guideline bullets

**Your answer:** Add concrete examples — **Correct**

Concrete examples anchor what "severe" actually means in practice, closing the interpretation gap that vague prose leaves open. More bullet points describing the same abstract concept just restates the ambiguity in different words. A second verification pass or lower temperature don't touch the root cause: the instruction itself is underspecified, not the model's execution of it.

## 6. Root-cause-to-mechanism matching (2/3) — Building with the Claude API / Prompt Engineering Techniques

**Q:** Two tags/terms in a prompt overlap in meaning (e.g. 'summary' and 'overview' used for different things) and Claude keeps mixing them up. What actually fixes this, versus what's a tempting-but-wrong default?

**Options:** Add verification pass / Increase temperature / Rewrite the terms / Wrap in XML tags

**Your answer:** Wrap in XML tags — **Wrong** (correct: Rewrite the terms)

Wrapping the terms in XML tags doesn't change what the terms *mean* — it just adds structural markup around two words that are already semantically overlapping. "Summary" and "overview" mixing up isn't a formatting/structure problem, it's a naming/definition problem: the fix is to rename them so they're actually distinct concepts, not to dress up the same overlapping words in tags.

This is the same root-cause-to-mechanism gap as question 5, but inverted: there, you correctly rejected the "add more description" default; here you reached for a structural fix (XML tags) when the actual fault was semantic (term overlap), not structural.

## 7. Root-cause-to-mechanism matching (3/3) — Building with the Claude API / Prompt Engineering Techniques

**Q:** Why is 'add a second model/verification pass' the wrong reflexive answer for a prompt-consistency problem caused by a vague or ambiguous instruction?

**Options:** Never useful / Doesn't fix root cause / Increases temperature / Too expensive

**Your answer:** Doesn't fix root cause — **Correct**

A verification pass adds cost and re-checks the output, but the underlying instruction is still ambiguous — so the second pass is judging against the same fuzzy criteria that caused the inconsistency in the first place. It's not "never useful" (verification has its place for other failure modes) and it's not about temperature or expense — it's that this specific fix treats a symptom while leaving the cause untouched.

## 8. Workflows vs. agents rule of thumb — Building with the Claude API / Agents and workflows

**Q:** What's the rule of thumb for choosing a workflow over an agent?

**Options:** Depends on speed / Prefer agents always / Workflow=single-step / Known vs unknown steps

**Your answer:** Known vs unknown steps — **Correct**

Workflows fit when the sequence of steps is known and fixed ahead of time (chaining, routing, parallelization) — you're hardcoding the control flow. Agents fit when the path isn't known upfront and Claude needs to use tools and its own judgment to figure out what to do next. It's not about step count or capability tier — the distinction is predictability of the path, not complexity or raw ability.

## 9. Hook exit codes — Claude Code in Action / Automating and verifying work

**Q:** What does a hook exiting with code 2 do?

**Options:** Success path / Ignored warning / Blocking error / Same as exit 1

**Your answer:** Blocking error — **Correct**

Exit code 2 is the "block and tell Claude why" signal — stderr gets fed back as context, and it blocks the action in almost every hook event. On the `Stop` event specifically it means "you're not actually done yet, keep going" rather than blocking a tool call. Exit 0 is success, exit 1 is a non-blocking error — code 2 is the one with teeth.

## 10. CLAUDE.md hierarchy — Claude Code in Action / Long-running sessions, steering, and CLAUDE.md configuration

**Q:** A consultancy works across 12 client projects. Each developer has personal preferences (editor keybindings), the firm has firm-wide coding standards, and each client project has its own conventions. What's the correct configuration architecture?

**Options:** Single root file / User-level for everything / User-level firm standards / Three-tier split

**Your answer:** Three-tier split — **Correct**

That maps each config tier to its actual scope: user-level `~/.claude/CLAUDE.md` for things personal to you (not shared, not version-controlled); project-level `CLAUDE.md` for anything the whole team/client project needs (checked into the repo); directory-level `CLAUDE.md` for rules scoped to a subsystem. A single mega-file or stuffing firm standards into the user-level file both break the "shared vs. personal" boundary this hierarchy exists to enforce.

## Weakest topics this session

- **tool-choice/tool-granularity confusion** — reflexively reached for "improve documentation" instead of recognizing overlapping tool responsibility needs redesign
- **root-cause-to-mechanism matching** — same defaulting pattern, this time reaching for a structural fix (XML tags) on a semantic problem (overlapping term meaning)

Both misses share a root pattern: when a failure mode has multiple candidate fixes (documentation vs. structural redesign vs. semantic rename), you're consistently under-weighting the structural/semantic fix in favor of "make the existing artifact more detailed."

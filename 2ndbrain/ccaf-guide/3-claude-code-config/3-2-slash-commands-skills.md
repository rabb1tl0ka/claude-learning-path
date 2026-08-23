# 3.2 Custom Slash Commands and Skills

## Source
https://claudecertificationguide.com/learn/3-claude-code-config/3-2-slash-commands-skills

## Summary

### Core System Architecture

Custom commands and skills have merged into a unified system with two file-structure paths that create identical `/command` invocations:

**Path Structures:**
- `.claude/commands/deploy.md` (flat file) creates `/deploy`.
- `.claude/skills/deploy/SKILL.md` (directory with SKILL.md entrypoint) also creates `/deploy`.

Critical distinction: "A skill is a **directory containing a `SKILL.md` file**" while "a command is a **flat Markdown file**." Placing a flat `.md` file directly in `.claude/skills/` does not create a command — it must use the directory structure.

The canonical (preferred) location is `.claude/skills/` because it supports features `.claude/commands/` does not: supporting-files directories alongside SKILL.md, automatic discovery when intent matches, and precedence when both share a name (skill wins). Both paths support identical YAML frontmatter and both are fully functional; `.claude/commands/` persists for backward compatibility.

### Scoping Levels: Project vs. User

**Project-Scoped (Shared via Git):**
- Location: `.claude/skills/` or `.claude/commands/` inside repository.
- Behavior: version-controlled, shared automatically when developers clone or pull.
- Use case: team-wide workflows like `/review`, `/deploy-check`, `/lint`, `/migration-guide`.

**User-Scoped (Personal):**
- Location: `~/.claude/skills/` or `~/.claude/commands/`.
- Behavior: personal, not version-controlled or shared.
- Use case: individual productivity workflows teammates don't need.

The documentation emphasizes a consistent pattern: "project-level (`.claude/`) is shared via git; user-level (`~/.claude/`) is personal. This applies to CLAUDE.md, commands/skills, and rules. Memorise this pattern — it appears throughout Domain 3."

### SKILL.md Frontmatter Configuration

Three critical YAML frontmatter options tested on the exam:

**`context: fork`**

"Runs the skill in an isolated sub-agent context. All the verbose output stays contained in the fork, and the main conversation stays clean."

Essential for:
- Codebase analysis (produces extensive file listings and code excerpts).
- Brainstorming (generates many alternatives and evaluations).
- Any task producing noisy, exploratory output.

Without it: "skill output flows into the main conversation and consumes context window tokens. For verbose skills, this degrades the quality of subsequent responses."

Example placement, for a `/analyse-feature` skill at `.claude/skills/analyse-feature/SKILL.md`:

```yaml
---
description: "Analyse a feature area of the codebase and report structure, patterns and risks"
context: fork
allowed-tools:
  - Read
  - Grep
  - Glob
argument-hint: "Provide a feature description or area of the codebase to analyse"
---
```

**`allowed-tools`**

"Pre-approves the listed tools so Claude can use them without a permission prompt while the skill is active."

Critical clarification: "It does not restrict which tools are available: every other tool remains callable, and your normal permission settings still govern anything that is not listed."

To actually remove tools from Claude's pool, use `disallowed-tools` instead or add deny rules in permission settings. The exam guide describes it as restricting access (the intended security boundary), though the current implementation pre-approves rather than restricts.

```yaml
allowed-tools:
  - Read
  - Grep
  - Glob
```

**`argument-hint`**

"Prompts the developer for required parameters when the skill is invoked without arguments. Improves the developer experience by making inputs explicit rather than relying on the developer to remember what the skill needs."

```yaml
argument-hint: "Specify the module path to analyse (e.g., src/api/auth)"
```

**Additional field: `description`**

Though not one of the "three critical" exam-tested fields, `description` performs critical work: "It's what Claude reads to decide whether a skill applies to what you just asked, so a skill without a useful one only ever runs when you type `/name` yourself."

Marked as **Recommended** in official documentation, `description` enables automatic skill discovery matching user intent. Without it, Claude falls back to the first paragraph of the skill body, typically an inferior summary.

### Skills vs. CLAUDE.md: The Critical Distinction

This distinction is "tested directly on the exam":

**Skills:**
- "on-demand, task-specific workflows."
- Descriptions remain always in context so Claude knows they exist, but full body loads only when invoked.
- Invocation can be explicit (`/skill-name`) or automatic when Claude matches the `description` to intent or when a `paths` field matches edited files.
- Skills with `disable-model-invocation: true` require explicit user invocation.

**CLAUDE.md:**
- "always-loaded, universal standards."
- Applied automatically to every session with no invocation step.
- Loads into context for every session (or every matching file for path-scoped rules).

**Rule:** "do not put task-specific procedures in CLAUDE.md. Do not put always-on reference material in skills."

**Example distribution:**
- API naming conventions that apply to every code generation task → CLAUDE.md or `.claude/rules/`.
- Multi-step codebase analysis workflow run occasionally → skill.
- Conventions for specific file types (e.g., test files) → path-scoped `.claude/rules/` (loads as always-on alongside matching files).

### Personal Skill Customisation

Developers can create personal variants in `~/.claude/skills/` (or `~/.claude/commands/`) with different names to avoid affecting teammates. Example: if the team has a standard `/analyse` skill but you prefer verbosity, create `~/.claude/skills/deep-analyse/SKILL.md`. Personal skills don't override or conflict with team versions.

### Quick Reference Table

| Need | Canonical Location | Also Works | Scoping |
|------|-------------------|------------|---------|
| Team-wide command | `.claude/skills/<name>/SKILL.md` | `.claude/commands/<name>.md` | Project (shared via git) |
| Team-wide command with frontmatter | `.claude/skills/<name>/SKILL.md` | `.claude/commands/<name>.md` | Project (shared via git) |
| Personal command | `~/.claude/skills/<name>/SKILL.md` | `~/.claude/commands/<name>.md` | User (not shared) |
| Universal standards | `.claude/CLAUDE.md` or root `CLAUDE.md` | — | Project (always loaded) |
| Personal preferences | `~/.claude/CLAUDE.md` | — | User (not shared) |

### Exam Traps Explicitly Called Out

**Trap 1: Flat File in `.claude/skills/`**
Error: creating `.claude/skills/review.md` and expecting a `/review` command.
Reality: "A skill is a directory containing a SKILL.md entrypoint (.claude/skills/review/SKILL.md); a flat .md file only creates a command under .claude/commands/."

**Trap 2: User-Scoped Team Commands**
Error: placing team-shared commands in `~/.claude/commands/` or `~/.claude/skills/`.
Reality: user-scoped paths are personal and not version-controlled. Team commands must go in project-scoped paths inside the repository.

**Trap 3: Skills as Always-On Guidance**
Error: thinking skills behave like CLAUDE.md for always-on guidance.
Reality: skills load on-demand as task-style workflows. While Claude can auto-invoke based on intent or path matching, they remain "separate invocation-style units rather than shaping every session by default." For always-on conventions, use CLAUDE.md or `.claude/rules/`.

**Trap 4: Forgetting `context: fork`**
Error: creating verbose analysis skills without isolation.
Reality: "context: fork isolates verbose skill output from the main conversation. Without it, brainstorming or codebase analysis output pollutes the context window."

**Trap 5: Task Workflows in CLAUDE.md**
Error: putting code review workflows, analysis routines, or brainstorming templates in CLAUDE.md.
Reality: CLAUDE.md is for always-loaded universal standards. Task-specific procedures belong in skills invoked on demand.

### Practice Scenario

Scenario: a team wants a `/review` command available to everyone who clones the repository. A developer also wants a personal `/brainstorm` skill that produces verbose codebase analysis output without cluttering the main conversation.

Correct Answer (Option C): "/review in .claude/commands/ for team sharing; /brainstorm as ~/.claude/skills/brainstorm/SKILL.md with context: fork frontmatter."

Why: project-scoped location ensures team access; user-scoped location keeps personal skill private; `context: fork` isolates verbose output.

### Build Exercise Summary

The exercise validates understanding through six tasks:

1. Create `.claude/commands/review.md` with team code review checklist (project-scoped, shared).
2. Create `~/.claude/skills/brainstorm/SKILL.md` with `context: fork` (user-scoped, isolated output).
3. Add `allowed-tools: [Read, Grep, Glob]` (read-only operations).
4. Add `argument-hint: "Provide a feature description or codebase area to explore"` (prompts for required params).
5. Verify `/review` appears for all project users; `/brainstorm` only for you (scoping boundary).
6. Invoke `/brainstorm` and confirm verbose output doesn't appear in main conversation (context: fork effect).

These directly test the scoping rules, frontmatter configuration, and context isolation mechanisms central to the exam.

# 3.3 Path-Specific Rules for Conditional Convention Loading

## Source
https://claudecertificationguide.com/learn/3-claude-code-config/3-3-path-specific-rules

## Summary

### Core Definition and Purpose

Path-specific rules apply conventions conditionally based on which files are being edited. They solve a specific problem: "conventions that must apply to one file type scattered across many directories." These rules live in the `.claude/rules/` directory and use YAML frontmatter with a `paths` field specifying glob patterns to determine when they activate.

### YAML Frontmatter Structure and Syntax

Path-specific rule files require frontmatter at the top:

```yaml
---
paths: ["pattern1", "pattern2"]
---
# Convention content follows
```

The `paths` field accepts an array of glob patterns as strings. Examples:
- `["terraform/**/*"]` — all files under terraform directory.
- `["**/*.test.tsx", "**/*.test.ts"]` — test files anywhere in codebase.
- `["src/api/**/*", "**/routes/**/*", "**/*.controller.ts"]` — API handler files across multiple locations.

### How Glob Patterns Work Across the Codebase

"A glob like `**/*.test.tsx` catches every test file in the codebase, wherever it sits." The double asterisk (`**`) matches across unlimited directory levels, while single asterisk (`*`) matches within one level. This allows one rule file to apply to files "scattered across 50+ directories" without needing duplicate CLAUDE.md files in each location.

### Key Distinction: Path-Specific Rules vs. Directory-Level CLAUDE.md

**Directory-level CLAUDE.md limitations:**
- Applies only to files in that specific directory.
- To cover test files across 50+ directories requires placing CLAUDE.md in every directory.
- Results in "50+ copies of the same conventions."
- Creates maintenance burden: "Every new directory with tests needs a new copy."
- Causes inevitable drift as updates lag across multiple files.

**Path-specific rules advantage:**
- One file with one pattern covers unlimited matching files across the entire codebase.
- Eliminates duplication and drift entirely.

### Token Efficiency: Path-Specific Rules vs. Root CLAUDE.md

Critical exam concept: "Path-scoped rules are more token-efficient than root CLAUDE.md because they load ONLY when editing matching files. This reduces irrelevant context and keeps the model focused on conventions that actually apply to the current work."

Practical impact: if Terraform conventions are in root CLAUDE.md, they "burn tokens even while you're editing React components." With path-specific rules, Terraform rules simply do not load during React editing sessions. In large projects with multiple convention categories, "this efficiency gain is substantial."

### When Rules Load and Don't Load

Rules activate automatically when the edited file matches any glob pattern in the `paths` array. Edit a file matching `terraform/**/*` and those rules load. Edit a React component and Terraform rules remain invisible. "The rules stay invisible until they're relevant."

Verification occurs via the `/memory` command, which shows which rule files are currently loaded in context.

### Practical Example Rule Files

**Testing conventions across the codebase:**
```yaml
---
paths: ["**/*.test.ts", "**/*.test.tsx", "**/*.spec.ts", "**/*.spec.tsx"]
---
# Test Conventions

- Use describe/it blocks with descriptive names that read as sentences
- Each test file must have at least one happy path and one error case
- Use factory functions for test data, not inline object literals
- Mock external services at the module boundary, not individual functions
- Assert behaviour, not implementation details
```

**API conventions:**
```yaml
---
paths: ["src/api/**/*", "**/routes/**/*", "**/*.controller.ts"]
---
# API Conventions

- All endpoints return { data, error, metadata } response shape
- Use Zod schemas for request validation at the handler boundary
- Log request ID on every error response
- Rate limiting configuration must be explicit, not inherited from defaults
```

**Infrastructure conventions:**
```yaml
---
paths: ["terraform/**/*", "**/*.tf", "infrastructure/**/*"]
---
# Infrastructure Conventions

- State files must reference remote backends, never local
- Use workspaces for environment separation
- Every module must be versioned with a CHANGELOG
```

### Decision Framework: Which Approach to Use

| Scenario | Best Approach |
|----------|---------------|
| Universal team standards applying to all code | Root CLAUDE.md |
| Conventions for one specific package directory | Directory-level CLAUDE.md |
| Conventions for file type spread across many directories | Path-specific rules with glob patterns |
| Task-specific workflows invoked on demand | Skills in .claude/skills/ |

### Critical Exam Traps and Misconceptions

**Trap 1: Choosing directory-level CLAUDE.md for cross-directory conventions**
When conventions must apply to files "spread across 50+ directories (like co-located test files), path-specific rules with glob patterns are correct. Directory-level CLAUDE.md would require placing a file in every directory — a massive maintenance burden."

**Trap 2: Placing file-type-specific conventions in root CLAUDE.md**
Storing Terraform conventions in root CLAUDE.md means they "consume tokens when editing React components." Path-specific rules preserve token budget by loading conditionally. This is framed explicitly as a token efficiency problem.

**Trap 3: Confusing skills with path-specific rules**
Both can auto-activate via paths frontmatter, but they serve different purposes: "Rules stay in context as background guidance — loaded when Claude reads a matching file — so they shape every edit. Skills load on-demand as task-style workflows, triggered either by the model's intent match or by explicit invocation." For "automatic, always-on convention loading for a file type, path-specific rules are the right answer."

### Exam Scenario and Answer

Practice scenario: a codebase has test files co-located with source files throughout 50+ directories. The team wants all tests to follow the same conventions.

Correct answer: Option B — create a rule file in .claude/rules/ with YAML frontmatter `paths: ["**/*.test.tsx", "**/*.test.ts"]` containing test conventions.

Why not the others:
- Option A (root CLAUDE.md): wastes tokens loading test rules during non-test work.
- Option C (CLAUDE.md in every directory): massive maintenance burden and drift risk.
- Option D (skill invoked manually): creates manual overhead; path-specific rules are automatic.

"The exam frequently presents the scenario of test files co-located with source files across many directories. The answer is always path-specific rules with glob patterns."

### Verification Mechanism: The /memory Command

The build exercise specifies verifying conditional loading using `/memory`:
- When editing a `.test.ts` file, `/memory` shows `.claude/rules/testing.md` as loaded, but not API or Terraform rules.
- When editing a file in `src/api/`, `/memory` shows `.claude/rules/api-conventions.md` as loaded, but not testing or Terraform rules.
- This proves the glob patterns correctly scope each rule file to its intended contexts.

Also suggested: comparing token footprints. "With all conventions in root CLAUDE.md, /memory shows the full set of conventions loaded even when editing a simple utility file. With path-specific rules, /memory shows only the relevant subset."

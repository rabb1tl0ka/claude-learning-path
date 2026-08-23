# 3.6 CI/CD Integration

## Source
https://claudecertificationguide.com/learn/3-claude-code-config/3-6-cicd-integration

## Summary

### Core Concept: Non-Interactive Execution

Claude Code defaults to interactive mode, requiring keyboard input. In CI pipelines with no human operator, this causes indefinite hangs. The `-p` flag (also `--print`) is the documented solution, switching Claude to print mode where it processes the prompt, outputs results to stdout, and exits.

Critical testing note: the exam identifies this as "Question 10 in the sample question set" and emphasizes it as "the single most directly tested item" in Domain 3.

**The -p Flag Mechanism**

Without `-p`:
```bash
# WRONG — hangs in CI
claude "Analyse this pull request for security issues"
```

With `-p`:
```bash
# CORRECT — runs non-interactively
claude -p "Analyse this pull request for security issues"
```

Common exam traps:
- `CLAUDE_HEADLESS=true` does not exist as an environment variable.
- `--batch` flag does not exist.
- Stdin redirection from `/dev/null` does not properly address Claude Code's interactive mode design.

### Structured Output for Machine Parsing

CI systems cannot read human-readable output. Two flags create machine-parseable results:

- **`--output-format json`**: wraps execution in a JSON envelope containing result text, session ID, and cost/usage metadata instead of human-readable format.
- **`--json-schema`**: validates the agent's final output against a provided JSON Schema (print mode only). Schema-conforming data lands in the envelope's `structured_output` field.

Complete example:
```bash
claude -p \
  --output-format json \
  --json-schema '{"type":"object","properties":{"findings":{"type":"array","items":{"type":"object","properties":{"file":{"type":"string"},"line":{"type":"integer"},"severity":{"type":"string"},"message":{"type":"string"}}}}}}' \
  "Review this PR for security issues"
```

Extraction method: use `jq '.structured_output'` to extract validated findings, not the top-level output. This enables downstream systems to post inline PR comments at exact file/line positions, filter by severity, and track findings across runs.

### Session Context Isolation

The problem: when Claude generates code within a session, it builds reasoning context — justifications for approach choices, considered tradeoffs, rejected alternatives. Asking it to review the same code in the same session causes it to retain this bias and be less likely to question prior decisions.

The solution: use a separate Claude Code invocation for review without shared session context:

```bash
# Step 1: Generate code (session A)
claude -p "Implement the authentication middleware"

# Step 2: Review code (session B — independent, no shared context)
claude -p "Review the authentication middleware for security issues, error handling gaps, and edge cases"
```

This "isn't a theoretical worry; it's a measurable effect," and connects to Domain 4 (multi-instance review architectures) and Domain 5 (context management).

### Incremental Review Context

Each automated review run on every push analyzes the entire PR from scratch, producing duplicate findings that developers already reviewed and chose not to address. This erodes developer trust when identical comments appear on every push regardless of fixes.

Solution implementation:
```bash
claude -p \
  --output-format json \
  "Review this PR. Here are the findings from the previous review:
  ${PREVIOUS_FINDINGS}

  Report ONLY:
  1. New issues not in the previous findings
  2. Issues from the previous findings that are still present

  Do NOT re-report previous findings the developer has already reviewed and chosen not to act on."
```

"Duplicate comments erode developer trust. If every push generates the same five comments regardless of whether the developer fixed the issues, developers stop reading the comments."

### CLAUDE.md for CI Context

When Claude Code runs in CI, it reads CLAUDE.md files identically to interactive execution. This file should contain:

- Testing standards (what makes valuable tests, patterns to follow/avoid).
- Available fixtures (which exist, usage methods, data contents).
- Review criteria (critical vs minor issue definitions).
- Existing test coverage (what's already covered, to avoid duplicates).

Example structure:
```
## Testing Standards

- Tests must use the factory pattern from test/factories/ for data creation
- Integration tests connect to the test database via test/setup/db.ts
- Do not test private implementation details — test public API contracts
- Coverage target: 80% branch coverage for new code
- Available fixtures: test/fixtures/users.json, test/fixtures/orders.json
```

"Without this context in CLAUDE.md, CI-invoked test generation produces low-value boilerplate. With it, generated tests follow the team's patterns and add genuine coverage."

### System Prompt Flags Reference

**Append vs Replace Distinction:**

| Flag | Effect |
|------|--------|
| `--system-prompt "<text>"` | Replaces entire default system prompt |
| `--system-prompt-file <path>` | Replaces default prompt with file contents |
| `--append-system-prompt "<text>"` | Appends text to default prompt |
| `--append-system-prompt-file <path>` | Appends file contents to default prompt |

Usage pattern: append when Claude should remain a coding assistant following additional rules (preserves default tool guidance and safety). Replace when identity/permission model differs from Claude Code's design (you own all prompt content).

### Headless Output and Limits Flags

| Flag | Effect |
|------|--------|
| `--output-format text\|json\|stream-json` | Output shape for `-p`; `json` and `stream-json` are machine-parseable |
| `--input-format text\|stream-json` | Input shape for `-p` |
| `--json-schema '<schema>'` | Schema-validated output for `-p`; with `--output-format json` it lands in envelope's `structured_output` field |
| `--max-turns <n>` | Cap agentic turns, then exit |
| `--verbose` | Full turn-by-turn output |

### Permissions, Tools, and Context Flags

| Flag | Effect |
|------|--------|
| `--permission-mode <mode>` | Start in `default`, `acceptEdits`, `plan`, `auto`, `dontAsk`, or `bypassPermissions` |
| `--allowedTools "<rules>"` | Tools that run without permission prompt, e.g. `"Bash(git diff *)" "Read"` |
| `--disallowedTools "<rules>"` | Deny rules; bare tool name removes tool entirely |
| `--tools "Bash,Edit,Read"` | Restrict which built-in tools are available |
| `--add-dir <path>` | Add directory Claude may read/edit (grants file access, not configuration discovery) |
| `--model <alias\|name>` | Set session model (`sonnet`, `opus`, or full model name) |

### Session and Startup Flags

- `-c` / `--continue`: resumes most recent conversation in current directory.
- `-r` / `--resume <id|name>`: resumes specific session.
- `--bare`: minimal mode skipping auto-discovery of hooks, skills, plugins, MCP servers, auto memory, and CLAUDE.md; provides only Bash and file read/edit tools; use when you want fast, predictable scripted runs without project configuration.

### Batch API vs Real-Time API Decision Boundary

| Workflow Type | API Choice | Reason |
|---|---|---|
| Pre-merge checks (blocking) | Real-time (synchronous) | Developers wait for results |
| Overnight technical debt reports | Batch API | Not time-sensitive, 50% savings |
| Weekly code audit | Batch API | Scheduled, latency-tolerant |
| Nightly test generation | Batch API | Runs overnight, reviewed next morning |

Critical exam trap (Sample Question 11): the Message Batches API offers 50% cost savings but has "processing times up to 24 hours with no guaranteed latency SLA." Pre-merge checks are blocking workflows where developers cannot merge until completion. The Batch API is unsuitable here due to lack of latency guarantees.

### Exact Exam Trap Summary

1. Hanging CI Pipeline: the fix is `-p` flag, not `CLAUDE_HEADLESS=true`, `--batch`, or stdin redirection.
2. Self-Review Effectiveness: same-session review is less effective than independent instances due to retained reasoning context bias.
3. Batch API for Pre-Merge: using Batches API for blocking CI checks ignores the 24-hour processing time with no SLA guarantee.
4. Prior Review Context: omitting previous findings causes duplicate comments on every push, eroding developer trust.

### Build Exercise Learning Outcomes

The exercise requires implementing:
1. CI script with `-p` flag for non-interactive PR analysis.
2. `--output-format json` with `--json-schema` for structured findings (file, line, severity, message).
3. JSON parsing to post inline PR comments at exact file/line numbers.
4. CLAUDE.md section documenting testing standards and CI-relevant context.
5. Two separate Claude invocations (generation and independent review).
6. Incremental review storing and reusing previous findings to eliminate duplicates.

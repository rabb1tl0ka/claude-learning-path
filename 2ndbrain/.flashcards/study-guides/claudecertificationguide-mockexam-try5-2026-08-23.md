# Study guide — claudecertificationguide-mockexam-try5-2026-08-23

Generated from imported exam misses. 7 questions missed; 1 mapped to an existing tracked topic, 6 are coverage gaps with no existing chapter or gap-topic note.

## Topic: hard-rules-belong-in-hooks

**Grounded in an actual missed question from this import** (also has prior weak-spot history: 1/2 wrong before this import, now 2/4 including this and one other historical miss).

**Core theory** (from `ccaf-learning/claude-code-in-action/long-sessions-and-steering.md` + `automating-and-verifying-work.md`):

- CLAUDE.md (and any `.claude/rules/` file) is **not enforced configuration** — it's instructions the model *reads and can drift from*, not code that runs. The longer or more rules-laden it gets, the less reliably any single rule is followed.
- If a rule is **hard and non-negotiable** ("never push to main", "every SQL migration must follow this naming pattern and include a rollback section"), it belongs in a **hook**, not a rules file — because a hook is code that executes deterministically, while CLAUDE.md/rules can only ask.
- Which hook type matters:
  - **PreToolUse** fires *before* the tool call executes — the right place to **block** an action outright (e.g. blocking a `delete_account` call until human approval, blocking a push to `main`).
  - **PostToolUse** fires *after* a tool call succeeds — the right place to **validate the actual output** against a rule (e.g. checking a just-created file's name matches a pattern, or that it contains a required section), because at that point the artifact exists and can be inspected.
- The tell in an exam scenario: language like "without relying on the model's judgment," "deterministic enforcement," or "100% compliance required" rules out any answer that's just a better-written rules file, better prompt wording, or more few-shot examples — those are all still probabilistic. Only a hook (pre- or post-, depending on whether you're blocking before or validating after) provides the guarantee.

**X vs Y**: a `.claude/rules/*.md` file (even a well-scoped, path-targeted one) is fundamentally the same category as CLAUDE.md — content the model reads and is expected to follow. A hook is not content — it's code. The exam consistently distinguishes "the model was told to do X" from "the system verified X happened," and only the second is deterministic.

**Self-check:**
1. A team wants Claude Code to never write directly to `main`. Should this go in CLAUDE.md or a hook — and which hook type?
2. A team wants every generated SQL migration file to be checked for a rollback section *after* it's written, without asking the model to remember to include one. Which hook type fires at the right moment to check the file that now exists on disk?

*(Answers: 1 — a PreToolUse hook, since it must block the action before it happens, not just discourage it. 2 — PostToolUse, since it needs to inspect the file's actual contents after creation, which doesn't exist yet at PreToolUse time.)*

## Coverage gaps (no existing chapter or gap-topic note)

These 6 misses don't map to any of the 4 course chapter notes or the existing `structured-data-extraction/`/`tool-choice-forcing/` gap-topic notes. They're real CCAF exam scope the notes simply haven't captured yet — not weak spots in material already studied. Listed here for visibility; not seeded into quiz history since there's nothing yet to attach them to.

1. **Escalation triggers overriding efficiency judgment.** *"A customer says: 'This is ridiculous, I have been waiting 20 minutes. Just connect me to a real person.' The agent has access to the customer's account and can see the issue is a simple password reset that takes 30 seconds. What should the agent do?"* — Correct answer: escalate immediately regardless of how simple the fix is; an explicit request for a human is a valid escalation trigger that overrides the agent's own judgment about efficiency.

2. **Sequencing already-known fixes with a dependency.** *"A developer has identified three issues in a function: two independent, one that changes the output shape the other two must conform to. How should feedback be sequenced?"* — Correct answer: fix the dependency-determining issue first, then address the independent ones individually — not an "ask clarifying questions" (interview) pattern, since nothing is actually unknown here.

3. **MCP tool description quality driving tool selection.** *"An agent ignores a sparsely-described CRM MCP tool ('CRM tool.') and falls back to Grep on local log files, producing incomplete results. What should the team do first?"* — Correct answer: expand the tool's own description to explain its actual capabilities, rather than patching the system prompt with a hard-coded instruction to prefer it.

4. **`.claude/rules/` glob scoping — catch-all globs defeat conditional loading.** *"A polyglot codebase has Terraform, Kubernetes, and Docker files across different directories. The team wants infrastructure-specific conventions to load only for the relevant file type, not for every session. What's the correct approach?"* — Correct answer: three separate path-scoped rule files, each with a glob targeting only its file type (e.g. `**/*.tf`, not `**/*`). **Note: this is the second exam in a row testing this exact trap** (try 4's Q50 had the same catch-all-glob miss) — worth prioritizing if a gap-topic note ever gets written for `.claude/rules/` mechanics.

5. **Multi-concern requests: decompose within one workflow vs. split across agents.** *"A customer reports being double-charged and also needs a billing address update. The agent resolves the address change but only partially investigates the double-charge. What architectural pattern fixes this?"* — Correct answer: decompose the request into its concerns, investigate each in parallel *with shared context*, then synthesize one unified resolution — not routing to two separate specialized agents, which would lose the shared account context both concerns need.

6. **Inter-step validation vs. prompt-quality fixes in a chained pipeline.** *"A 3-step extraction pipeline (extract text → classify document type → extract fields by type) occasionally misclassifies documents in step 2, corrupting step 3's output. The team proposes combining steps 2 and 3 into one prompt. What's the better fix?"* — Correct answer: keep the steps separate and add an explicit validation check between step 2 and step 3 — more few-shot examples in step 2 might reduce the error rate but can't guarantee catching every misclassification before it cascades.

## Suggested study order

1. Re-read `long-sessions-and-steering.md`'s CLAUDE.md section and `automating-and-verifying-work.md`'s hook-lifecycle section side by side, specifically the Pre/PostToolUse distinction — this is the one topic here with real prior history (2 misses across 4 attempts now).
2. Skim the 6 coverage-gap items above — no note to study from yet, but worth recognizing the shape of each ("explicit trigger overrides efficiency," "shared-context decomposition beats agent-splitting," "structural checkpoint beats better prompting") since these patterns recur across mock exams even without a dedicated note.
3. If gap-topic notes get written later for any of these (especially #4, the repeat `.claude/rules/` glob trap), this study guide should be revisited.

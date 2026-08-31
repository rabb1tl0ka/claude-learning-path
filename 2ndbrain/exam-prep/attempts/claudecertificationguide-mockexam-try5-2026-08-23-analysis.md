## Overall result

**884/1000 — PASSED** (pass mark 720). 53 of 60 correct (88%). This raw score is the number closest to real CCAF readiness (per `exam-prep/CLAUDE.md`) — the course/gap mapping below is routing info, not a competing readiness metric. Slightly below try 4 (895/1000, 90%) — the first time a try-N score has dropped rather than improved.

A Gemini transcript of Bruno taking this attempt out loud (Google Doc, "Claude Code Learning Path - Practice 1") was added after the initial analysis and is now folded in below — it sharpens the *why* behind several misses beyond what the exam's own explanation text shows.

## Domain breakdown

| Domain | Score | Note |
|---|---|---|
| D1 Agentic Architecture & Orchestration | 13/14 (93%) | |
| D2 Tool Design & MCP Integration | 10/11 (91%) | |
| D3 Claude Code Configuration & Workflows | 9/12 (75%) | — weakest |
| D4 Prompt Engineering & Structured Output | 11/12 (92%) | |
| D5 Context Management & Reliability | 10/11 (91%) | |

## Failure patterns

**1. Reaching for a softer / more elaborate fix instead of the deterministic or already-implied structural one (Q36, Q40, Q57, Q58)** — the dominant pattern, 4 of 7 misses, and a direct repeat of try 4's #1 pattern ("reaching for more machinery instead of the intended structural/judgment fix"):

- **Q36**: a team needs SQL migration filenames and rollback sections enforced *without relying on the model's judgment*. Picked a `.claude/rules/migrations.md` context file; the correct answer was a **PostToolUse hook** that validates the filename pattern and checks for a rollback section after the file is written. A rules file is still just instructions the model reads and can drift from — only a hook runs deterministic code against the actual output. Notably, hook mechanics themselves weren't the gap: Q3, Q11, Q34, and Q43 (all genuine PreToolUse/PostToolUse hook questions) were answered correctly. **The transcript catches the actual failure live**: Bruno explicitly considered the PostToolUse hook and talked himself out of it — "post[tool use] validates and checks for the rollback... this will only validate, wouldn't do it" — then picked the rules file instead. The gap isn't "doesn't know hooks exist," it's **equating "validate" with "not enforce."** He was reasoning from a mental model where enforcement means blocking/preventing, so a hook that only checks-after-the-fact didn't register as a real safeguard, even though the question only asked for deterministic detection, not prevention.
- **Q40**: an agent underuses a sparsely-described CRM MCP tool and falls back to Grep. Picked adding a system prompt instruction ("always use CRM tool, never Grep"); the correct fix was **expanding the tool's own description** to explain its actual capabilities. The transcript shows this was a fast, low-deliberation pick — "Oh, it's B for sure" — with no visible weighing of the description-fix option, unlike most other questions where Bruno talks through each option in turn. A system-prompt patch is brittle and doesn't scale if the tool set changes; fixing the description fixes tool selection at the source, the same "fix the source, don't patch downstream" logic seen with Q36.
- **Q57**: a customer's double-charge-plus-address-change request gets only partially resolved. Picked routing to two separate specialized agents (billing disputes / address changes); the correct answer was **decomposing the request into items within one workflow, investigating in parallel with shared context, then synthesizing one resolution**. The transcript shows Bruno directly weighed and rejected the "shared context" framing — "we don't need sh[ared] context here" — when shared context was exactly the point: both concerns (a wrongful double-charge and an address change) live on the same customer account, and splitting them into separate agents loses that shared account state. This sharpens the miss from "added more agents than needed" to a specific misjudgment that the two concerns were context-independent when they weren't.
- **Q58**: a three-step extraction pipeline occasionally misclassifies document type in step 2, corrupting step 3. Picked adding more few-shot examples to step 2's prompt; the correct answer was **keeping the steps separate and adding an explicit validation check between step 2 and step 3**. The transcript shows this pick was quick and confident with no hesitation ("I would go with C here, man"), echoing Q13/Q28/Q42's correctly-answered retry-boundary logic elsewhere in the exam (a prompt-quality fix isn't the same as a missing safeguard) — the concept was clearly available to him, just not retrieved here.

The common thread across all four: each scenario contained an explicit constraint or tell ("without relying on model judgment," "occasionally," "still occasionally calls") pointing at a structural/deterministic gap, and the instinct was to reach for a prompt-level or agent-count fix instead. The transcript adds a sharper mechanism for at least two of these (Q36, Q57): not a knowledge gap, but a live moment of reasoning *toward* the correct answer's core idea (validation-as-enforcement, shared-context-as-necessary) and then rejecting it.

**2. Path-scoped conditional-loading mechanics — the same catch-all-glob trap as try 4 (Q45)** — a DevOps team wants Terraform/Kubernetes/Docker conventions to load only for their respective file types, without loading for every session. Picked a single `.claude/rules/infrastructure.md` with `paths: ["**/*"]`; the correct answer was **three separate path-scoped rule files** (`terraform.md`, `kubernetes.md`, `docker.md`), each with its own targeted glob. `**/*` matches every file regardless of type, which makes the "conditional" rule behave exactly like an always-loaded root CLAUDE.md — defeating the entire premise of path-specific rules. This is the *identical* trap as try 4's Q50 (there: a root CLAUDE.md with the same effective always-on behavior). Two attempts in a row now show a specific blind spot for how `.claude/rules/` glob scoping actually works, not just a general "know rules exist" gap — worth a dedicated look at this mechanic specifically.

**3. Reasoning quality thinning in the back third of the exam (Q45, Q57, Q58, and to a lesser extent Q40)** — a pattern only visible from the transcript, not the exam text. Through roughly Q1-Q35, Bruno's narration walks through each option individually with reasons for ruling each out. From around Q43 onward the transcript compresses sharply into short, low-detail exclamations — "Yeah, it's been 24 35. I think it's seaman 46. I like C7. I like me. 48 It's 49. Maybe." — and stays that way through the rest of the exam. Three of the seven misses (Q45, Q57, Q58) and one borderline case (Q40's snap pick) fall inside or near this compressed zone. This doesn't prove time pressure caused the misses, but it's consistent with same-mechanic-repeat errors like Q45 (identical glob catch-all trap as try 4's Q50) surfacing exactly where deliberation visibly dropped off — worth watching on the next attempt: if the back-third compression recurs, it's a pacing issue, not a knowledge gap, and the fix is slowing down late-exam rather than more content review.

## Other misses

One miss didn't fit either pattern cleanly:

- **Q32**: a developer has already identified three issues in one function — two independent, one that determines the shape the other two must conform to. Picked the "interview pattern" (ask Claude clarifying questions about all three before changing anything); the correct answer was to **fix the dependency-determining issue first, then address the two independent ones**. The interview pattern is for undiscovered unknowns; here the developer already knew everything needed — the question was about sequencing already-known work, not discovery. The transcript shows no hesitation on this one either ("Yeah, I like option D") — a confident but reflexive application of a memorized heuristic (interview pattern) triggered by surface similarity ("multiple issues, get feedback") rather than checking whether its actual precondition (unknown unknowns) applied.

## Escalation-trigger override (Q29)

A customer explicitly says "connect me to a real person," and the underlying issue (a password reset) is trivially fixable in 30 seconds. Picked resolving it quickly and explaining escalation is unnecessary; the correct answer was to **escalate immediately regardless of how simple the fix is**, because an explicit request for a human is one of the stated valid escalation triggers and overrides the agent's own judgment about efficiency.

The transcript shows Bruno actually noticed the tension and reasoned past it: he considered that the customer might have been waiting 20 minutes already, decided "we have nothing to do with it, the problem is not the agent," and picked the efficiency-first option anyway — "I'm reading option A and it looks pretty good... escalation is unnecessary." This isn't a missed concept; it's **prioritizing solving the problem well over honoring an explicit boundary the user stated**, the same shape of error as Q36/Q57 above (reasoning toward a good-sounding practical fix while overriding an explicit constraint in the prompt). Worth flagging because it's a values-ordering issue (efficiency vs. explicit user instruction) more than a technical gap — the kind of thing that's easy to keep missing precisely because the "wrong" answer feels like the more helpful one.

## Strength patterns

**Human review & confidence calibration (D5) — 6/6 correct across five distinct scenarios (Q12, Q19, Q21, Q27, Q44, Q54).** Aggregate-metrics-trap, stratified-sampling, and field-level-calibration concepts were each tested multiple times in different framings (contract extraction, legal documents, financial reports) and answered correctly every time, including a select-3 question (Q54) that required identifying all three safeguards at once — strong evidence this cluster is genuinely internalized, not pattern-matched once.

**Deterministic-enforcement hook mechanics, in isolation — 4/4 correct (Q3, Q11, Q34, Q43).** Every question that was purely "which hook type and when does it fire" (PreToolUse for blocking before execution, PostToolUse for validating after) was answered correctly. The nuance from Failure Pattern 1 above is that recognizing *when a scenario calls for a hook at all* (Q36, vs. a rules file) is the actual gap, and specifically not trusting a validate-only hook as sufficient enforcement, not the hook mechanics themselves once "use a hook" is already on the table.

## Coverage-gap note (informational)

31 of 60 questions test material that doesn't map to any of the 4 course notes or an existing gap-topics note — again a substantial share of real CCAF scope the courses don't teach directly. Bruno went 25/31 (81%) on these, noticeably below his 94% on course-mapped questions (17/18) and 100% on gap-mapped questions (11/11) — this uncovered-material bucket is where the real headroom is, more than either of the two things already being tracked. Clusters worth naming:

- **Tool interface design & MCP structured error responses** (tool descriptions, boundary descriptions, transient/business/access-failure error categories, MCP server description quality) — the entire D2 domain (11/11 questions) falls here; 10/11 correct, only Q40 missed. Strong performance despite zero existing notes, but also zero paper trail if this drifts.
- **Multi-agent orchestration & workflow enforcement specifics** (isolation, hub-and-spoke, subagent invocation via the Task tool, confidence-based escalation, session state/resumption, multi-concern decomposition) — Q14, Q20, Q25, Q31, Q39, Q57 (5/6 correct, only Q57 missed).
- **Path-specific `.claude/rules/` glob mechanics** — Q16, Q45 (1/2 correct, see Failure Pattern 2, and the identical miss on try 4's Q50).
- **CI/CD session isolation** (independent Claude Code invocations per pipeline step to avoid shared reasoning bias) — Q5, Q18 (2/2 correct).
- **Context window management specifics** (persistent facts blocks, token budget allocation) — Q47, Q49 (2/2 correct).
- **Escalation/ambiguity edge cases beyond explicit gap-note coverage** — Q9, Q29, Q37 (2/3 correct, Q29 missed).
- **Message Batches API, nullable schema fields, multi-pass review** — Q4, Q33, Q59 (3/3 correct).

## Course-topic performance

Grouped by chapter note. Routing info from the exam's tested concepts matched to existing course notes by subject matter, not by the exam's own third-party lesson labels.

| Chapter note | Score | Wrong |
|---|---|---|
| [long-sessions-and-steering.md](../../ccaf-learning/claude-code-in-action/long-sessions-and-steering.md) | 4/5 (80%) — weakest | Q36 |
| [automating-and-verifying-work.md](../../ccaf-learning/claude-code-in-action/automating-and-verifying-work.md) | 3/3 (100%) | — |
| [sharing-and-scaling-claude-code.md](../../ccaf-learning/claude-code-in-action/sharing-and-scaling-claude-code.md) | 2/2 (100%) | — |
| [structured-data.md](../../ccaf-learning/claude-api/accessing-claude-with-the-api/structured-data.md) | 1/1 (100%) | — |
| [prompt-engineering-techniques.md](../../ccaf-learning/claude-api/prompt-engineering-techniques/prompt-engineering-techniques.md) | 2/2 (100%) | — |
| [multi-turn-conversations-with-tools.md](../../ccaf-learning/claude-api/tool-use-with-claude/multi-turn-conversations-with-tools.md) | 2/2 (100%) | — |
| [agents-and-workflows.md](../../ccaf-learning/claude-api/agents-and-workflows/agents-and-workflows.md) | 3/3 (100%) — strongest (tied) | — |

## Gap-topic performance

Informational routing only, which existing gap-topics note this maps to, not a separate readiness metric. These questions count toward the overall score like any other, since gap topics are real CCAF exam scope.

| Gap-topic note | Score | Wrong |
|---|---|---|
| [retry-with-error-feedback.md](../../ccaf-learning/structured-data-extraction/retry-with-error-feedback.md) | 3/3 (100%) | — |
| [semantic-vs-schema-validation.md](../../ccaf-learning/structured-data-extraction/semantic-vs-schema-validation.md) | 1/1 (100%) | — |
| [stratified-sampling.md](../../ccaf-learning/structured-data-extraction/stratified-sampling.md) | 3/3 (100%) | — |
| [aggregate-metrics-trap.md](../../ccaf-learning/structured-data-extraction/aggregate-metrics-trap.md) | 3/3 (100%) | — |
| [confidence-calibration.md](../../ccaf-learning/structured-data-extraction/confidence-calibration.md) | 3/3 (100%) | — |
| [structured-claim-source-mapping.md](../../ccaf-learning/structured-data-extraction/structured-claim-source-mapping.md) | 1/1 (100%) | — |
| [tool-choice-forcing.md](../../ccaf-learning/tool-choice-forcing/tool-choice-forcing.md) | 1/1 (100%) | — |

## Next step

Run `/flashcards import exam-prep/attempts/claudecertificationguide-mockexam-try5-2026-08-23.pdf` from `2ndbrain/` to seed the missed questions into weak-spot tracking and generate a targeted study guide.

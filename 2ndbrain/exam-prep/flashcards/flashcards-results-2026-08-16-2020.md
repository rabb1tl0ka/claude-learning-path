# Flashcards results — 2026-08-16 20:20

Scope: `introducing-tool-use.md`, `prompt-engineering-techniques.md`, `agents-and-workflows.md`, `automating-and-verifying-work.md` — the four course chapters flagged as weak by `claudecertificationguide-mockexam-try1-2026-08-16-analysis.md`.

**Score: 6/10**

## Q1 — Introducing Tool Use / tool description quality — ❌ Wrong

> Claude is repeatedly calling the wrong tool out of two similarly-named tools in a set. Per the course's guidance on tool descriptions, what's the most direct fix?

- Rewrite descriptions to be more distinct about what each does, when to use it, what it returns ✅ (correct)
- Add a second Claude call that double-checks which tool was picked
- Lower the temperature so tool selection is more deterministic
- Add more few-shot examples of correct tool calls to the system prompt ← *your answer*

**Teaching:** The chapter's tool-schema guidance says Claude picks a tool almost entirely off its `name`/`description` — a 3-4 sentence spec covering what it does, when to use it, and what it returns. If two tools get confused, that's a symptom that the descriptions don't clearly separate their scope. Few-shot examples patch *behavior* for the phrasings you happened to show; they don't fix the underlying spec ambiguity, so the same confusion resurfaces the moment a new phrasing hits the same gap. Rewriting the descriptions fixes the actual signal Claude is selecting on.

## Q2 — Introducing Tool Use / tool-choice/tool-granularity confusion — ❌ Wrong

> Two tools overlap in what they can do (e.g. one sets a reminder for "a date" and another for "a duration from now") and Claude picks the wrong one under time pressure. What's actually wrong here?

- The tools have overlapping responsibility/granularity — redesign so each owns a distinct, non-overlapping job ✅ (correct)
- Claude needs a few-shot example added showing the correct tool for each case ← *your answer*
- A verification pass should check which tool was called and retry if wrong
- The descriptions need to be longer and more detailed

**Teaching:** This is a different failure mode from Q1 — it's not that the wording is unclear, it's that the two tools' actual *jobs* overlap (both could plausibly apply to the same request). No description, however precise, and no example can resolve a design where two tools are drawn to cover the same ground. The fix is architectural: redraw the tool boundaries so each tool owns one distinct piece of the problem space (this is the same principle behind "tools should be abstract, not hyper-specialized" from the Agents and Workflows chapter — the issue there is granularity, not wording).

## Q3 — Prompt Engineering Techniques / root-cause-to-mechanism matching — ❌ Wrong

> A rubric criterion is written as vague prose (e.g. "flag severe issues") and Claude's judgments are inconsistent across similar cases. What's the correct fix?

- Add a second model/verification pass that re-checks the first judgment for consistency
- Replace the vague prose description with concrete examples showing what counts as each severity level ✅ (correct)
- Add more guideline bullet points describing the general concept in more detail ← *your answer*
- Lower the temperature to make output more deterministic

**Teaching:** The chapter's "providing examples" technique (one-shot/multi-shot) exists specifically for this: ambiguity that abstract prose can't pin down. A concrete example shows Claude exactly where the line falls for each severity level, in a way no amount of additional prose can. Adding more guideline bullets just restates the same abstract concept in more words — it's still prose, still lacks a concrete instance to calibrate against, so the inconsistency doesn't actually go away.

## Q4 — Prompt Engineering Techniques / root-cause-to-mechanism matching — ✅ Correct

> Two tags/terms in a prompt overlap in meaning (e.g. "summary" vs "overview" used for different things) and Claude keeps mixing them up. What actually fixes this?

**Your answer:** Rewrite the terms so they're non-overlapping and distinct ✅

**Teaching:** Right call — the ambiguity lives in the terms themselves, not in how strongly they're enforced. A verification pass would just re-run the same confusable vocabulary and hope for a cleaner result; XML-wrapping without renaming leaves the actual collision in place. Renaming to genuinely distinct terms removes the ambiguity at the source.

## Q5 — Prompt Engineering Techniques / root-cause-to-mechanism matching — ✅ Correct

> Why is "add a second model/verification pass" the wrong reflexive answer for a prompt-consistency problem caused by a vague or ambiguous instruction?

**Your answer:** It doesn't fix the root cause (the ambiguity itself) — it just re-runs the same ambiguous instruction and hopes for a different, checked result, adding cost without resolving why the output varies ✅

**Teaching:** Exactly the mechanism — a verification pass is a *check*, not a *fix*. It can catch some bad outputs after the fact, but the same ambiguous instruction still produces inconsistent judgments on the first pass; the underlying cause (vague criterion) is untouched. This is the general shape of your exam-analysis pattern 1: the correct move is almost always to fix the actual root cause (rewrite the ambiguous thing), not bolt on a second look.

## Q6 — Agents and Workflows / multi-agent context-passing ownership — ✅ Correct

> In a multi-agent pipeline, a downstream/receiving agent produces a wrong result because it never got the structured context it needed. Whose responsibility was that?

**Your answer:** The coordinator/upstream agent's — it's responsible for passing structured context forward to the agent that needs it, not just handing off a bare task ✅

**Teaching:** Right. The chapter frames orchestration (chaining, routing, coordinator/subagent delegation) as the upstream step owning what gets handed downstream — a subagent can only be as good as the context it's given. A downstream agent that's "working correctly" on the input it received can still produce a broken result if that input was incomplete; blaming the receiving agent misdiagnoses where the actual break happened.

## Q7 — Agents and Workflows / chaining vs routing — ✅ Correct

> Claude is given a long prompt with many constraints and keeps partially ignoring some of them. Which workflow pattern actually fixes this, and why is it not routing?

**Your answer:** Chaining — break the task into sequential steps and send narrow follow-up corrections; routing doesn't apply because there's nothing to classify/categorize here ✅

**Teaching:** Right, and this is a direct echo of your own chapter note — you got this exact scenario wrong on the original course quiz for the same reason (reaching for routing/categorization when there's nothing to categorize). Chaining works because a long prompt with many constraints Claude partially misses isn't a classification problem — it's a "too much for one call" problem, better solved by narrowing scope per step than by sorting the input into a category.

## Q8 — Automating and Verifying Work / risk-matched enforcement (hooks vs prompts) — ✅ Correct

> A team wants to make sure Claude never force-pushes to a shared branch. Which mechanism actually guarantees this, and why not just a CLAUDE.md rule?

**Your answer:** A hook — because CLAUDE.md and skills are instructions Claude follows and can still be skipped/forgotten, while a hook is code that runs and can outright block the action ✅

**Teaching:** Right — this is the chapter's three-way split. CLAUDE.md and skills shape behavior Claude *chooses* to follow; a hook is deterministic code that fires regardless of what Claude decides, so it's the only one of the three that can actually *guarantee* an outcome like "never force-push." The corollary (from your exam miss on Q53) is that this guarantee should be reserved for genuinely risk-critical rules — hooking everything "for consistency" over-applies hard enforcement where a CLAUDE.md convention would do fine.

## Q9 — Introducing Tool Use / multi-block assistant messages — ❌ Wrong

> When Claude decides to use a tool, what does the assistant message actually contain?

- Potentially multiple content blocks — e.g. a text block plus a tool_use block with the tool's id, name, and input ✅ (correct)
- Only a single tool_use block, since Claude cannot emit text on a tool-call turn
- A single plain-text string describing which tool to call
- A tool_result block confirming the tool already ran ← *your answer*

**Teaching:** This mixes up two different turns in the exchange. When Claude *decides* to call a tool, that's Claude's own output — an assistant message that can contain a text block (e.g. "Let me find that for you") plus a `tool_use` block. The `tool_result` block is the *opposite direction*: it's what the developer sends back to Claude on the *next* turn, after the function actually ran, carrying the output back in. Getting this backwards would break the message-history reconstruction the chapter stresses — you have to preserve the full block structure, not just extract text, precisely because these blocks flow in specific directions across turns.

## Q10 — Introducing Tool Use / sending tool results — ✅ Correct

> After a tool function runs, how is its output returned to Claude in the next turn?

**Your answer:** As a tool_result block referencing the original tool_use id ✅

**Teaching:** Right, and this is the natural counterpart to Q9 — the `tool_result` block carries the actual output back to Claude, tagged with the same id as the `tool_use` block that requested it, so Claude can match the result to the call it made.

## Weakest topics this session

- **Introducing Tool Use** — 2/4 wrong (tool description quality, multi-block assistant messages) — still the weakest chapter, matching its 0/2 on the real mock exam.
- **Prompt Engineering Techniques** — 1/3 wrong on root-cause-to-mechanism matching (vague-prose → concrete-examples vs. "more guidelines").
- Agents and Workflows, Automating and Verifying Work — both 1/1 correct this session.

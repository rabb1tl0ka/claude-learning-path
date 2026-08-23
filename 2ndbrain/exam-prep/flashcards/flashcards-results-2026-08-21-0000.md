# Flashcards results — 2026-08-21

**Scope:** Prompt Engineering Techniques (targeted at weak topic `few-shot-vs-tool-granularity`)
**Score:** 9/10

## Q1 — Root-cause-to-mechanism matching

**Question:** A rubric criterion is written as vague prose (e.g. 'flag severe issues') and Claude's judgments are inconsistent across similar cases. What's the correct fix?

**Options:**
- Lower the temperature to make output more deterministic
- Replace the vague prose description with concrete examples showing what counts as each severity level (✅ correct, chosen)
- Add more guideline bullet points describing the general concept in more detail
- Add a second model pass that re-checks the first judgment for consistency

**Result:** Correct.

**Explanation:** Vague prose ("severe") is the root cause — Claude has no anchor for what counts as severe vs. not, so it drifts case to case. Concrete labeled examples give it a fixed reference point. Neither a verification pass nor lower temperature touches the actual ambiguity; they just re-run or dampen the same underspecified rule.

## Q2 — Being clear and direct

**Question:** Per the 'being clear and direct' technique, what should the first line of a prompt look like?

**Options:**
- A polite question asking Claude if it's willing to help
- An XML tag wrapping the entire prompt
- A direct, command-style instruction with an action verb (e.g. 'write', 'generate') rather than a question (✅ correct, chosen)
- A long paragraph of background context before any instruction

**Result:** Correct.

**Explanation:** "Clear and direct" means leading with the actual instruction as a command ("Write...", "Generate...") rather than burying it in politeness or background — Claude follows explicit imperatives more reliably than an implicit ask.

## Q3 — Being specific

**Question:** An open-ended prompt like 'write a story' produces wildly inconsistent length and structure across runs. Which technique directly addresses this?

**Options:**
- Lower max_tokens to force a shorter output
- Being specific — add guideline lists (length, structure, required elements) or step-by-step instructions to constrain the output space (✅ correct, chosen)
- Provide a single one-shot example only
- Structure with XML tags around the word 'story'

**Result:** Correct.

**Explanation:** "Write a story" leaves the entire output space open, so results vary wildly. Being specific narrows that space directly — length, structure, required beats — which is a different lever from XML tags (organizing existing content) or examples (showing one instance rather than constraining the range).

## Q4 — Structuring with XML tags

**Question:** When should you wrap interpolated content (e.g. 20 pages of sales records) in a descriptive XML tag like <sales_records>?

**Options:**
- Only when the content contains numbers
- When a prompt interpolates a large chunk of content, so Claude can clearly parse what that chunk represents (✅ correct, chosen)
- XML tags are required on every prompt regardless of content
- Only when using the extended thinking feature

**Result:** Correct.

**Explanation:** XML tags are a structural aid for large interpolated blocks — they mark where a chunk starts/ends and label what it is, so Claude doesn't have to guess where the "sales records" end and your instructions begin. It's not a universal requirement or tied to any specific feature like extended thinking.

## Q5 — Providing examples

**Question:** What's the distinction between one-shot and multi-shot prompting?

**Options:**
- They're the same technique, just different names
- One-shot gives a single example; multi-shot gives multiple examples, useful for covering edge cases (✅ correct, chosen)
- One-shot is for text tasks, multi-shot is only for code generation tasks
- One-shot means zero examples; multi-shot means exactly one example

**Result:** Correct.

**Explanation:** One-shot = one example, multi-shot = several. Multi-shot's value is showing a spread — different formats, edge cases, tricky inputs — so Claude generalizes the pattern rather than just copying a single instance.

## Q6 — Prompt engineering workflow

**Question:** What was the described workflow for improving a prompt in this chapter's project?

**Options:**
- Write the best possible prompt on the first attempt and never re-evaluate
- Set a goal, write a deliberately weak initial prompt, evaluate it, apply a technique, re-evaluate — repeating per technique (✅ correct, chosen)
- Ask Claude to write its own prompt from scratch with no human input
- Only apply XML tags, since that technique alone maximizes eval score

**Result:** Correct.

**Explanation:** The workflow is iterative and evidence-driven: set a goal, deliberately start weak, measure it with an eval, apply one technique at a time, and re-measure — so you can attribute improvement to a specific change rather than guessing which of several simultaneous tweaks helped.

## Q7 — Few-shot-vs-tool-granularity ⚠️ MISSED

**Question:** A tool called analyze_content with the description 'Analyses content from various sources' is used indiscriminately for web scraping, document parsing, and code analysis, leading to poor results for each. What is the most effective fix?

**Options:**
- Add few-shot examples demonstrating correct use for each of the three cases
- Lower the temperature so the agent applies the tool more consistently
- Split analyze_content into purpose-specific tools (e.g. scrape_web, parse_document, analyze_code), each scoped to one job (✅ correct)
- Rewrite the tool's description to be more detailed about all three use cases (❌ chosen)

**Result:** Wrong.

**Explanation:** The correct answer is splitting `analyze_content` into purpose-specific tools. `analyze_content` isn't ambiguously worded, it's doing three unrelated jobs at once. No amount of description detail fixes that — Claude still has to guess which "mode" of the tool applies each time, and a longer description just gives it more ways to guess wrong. The fix is architectural: split by job, not by wording. This is a reflexive-but-wrong default: "improve the description" is the right move for *unclear* wording between genuinely distinct tools (see the earlier "tool description quality" topic), but the wrong move when the real problem is *scope* — one tool covering ground that should be several. This is the 4th consecutive miss on this topic — now a confirmed weak spot, not a fluke.

## Q8 — Prompt-clarity-vs-verification-patch

**Question:** A CI/CD code review pipeline has a 40% false positive rate on documentation-mismatch findings, causing developers to ignore all review categories. What is the most effective fix?

**Options:**
- Lower the model's temperature to reduce false positives
- Add a second verification pass that re-checks each documentation-mismatch finding before it's shown to developers
- Rewrite the documentation-mismatch criterion so it's specific enough to stop over-flagging, rather than patching around it downstream (✅ correct, chosen)
- Remove the documentation-mismatch category entirely so developers stop ignoring the rest

**Result:** Correct.

**Explanation:** This is the root-cause-to-mechanism pattern again: a vague/over-broad criterion is generating false positives, so tighten the criterion itself. Adding a verification pass or removing the category entirely just patches the symptom (developer trust erosion) without fixing why the criterion over-flags in the first place.

## Q9 — Concrete-examples-vs-vague-prose

**Question:** A code review prompt classifies severity using prose descriptions like 'critical means the code is dangerous,' and judgments are inconsistent. What is the most effective improvement?

**Options:**
- Add a second model pass that re-scores severity for consistency
- Lower the temperature to make severity scoring more deterministic
- Replace the vague prose severity definitions with concrete labeled examples of what counts as each severity level (✅ correct, chosen)
- Expand the prose definition with more adjectives describing danger

**Result:** Correct.

**Explanation:** Same fix as Q1: "dangerous" is subjective and unanchored, so concrete labeled examples give Claude a fixed reference for what actually counts as each severity tier, rather than more adjectives that just describe the same fuzzy concept in different words.

## Q10 — Prompt-keyword-overlap

**Question:** A system prompt defines two review categories: 'Check for security vulnerabilities in each function' and 'Check for performance issues in each loop.' The model frequently calls the wrong tool. What is the root cause and best fix?

**Options:**
- The model needs a lower temperature to reduce misclassification
- The instructions need to be wrapped in XML tags to disambiguate them
- The two instructions use overlapping/similar phrasing that doesn't clearly map to distinct tools — rewrite them with distinct, non-overlapping keywords tied to each tool's actual scope (✅ correct, chosen)
- The tools' descriptions need more few-shot examples of correct calls

**Result:** Correct.

**Explanation:** "Security vulnerabilities in each function" and "performance issues in each loop" both use generic instructional phrasing without a strong lexical anchor to a specific tool, so the fix is tightening the wording itself to map cleanly to each tool's scope, not adding examples or formatting.

## Weakest topics this session

- **few-shot-vs-tool-granularity** (missed) — now 4/4 wrong all-time, confirmed weak spot. Core confusion: treating an overloaded, multi-job tool as a wording/description problem instead of a scope/granularity problem. Distinguish from "tool description quality" (fix = rewrite wording, applies when tools are genuinely distinct but confusingly worded) and "tool-choice/tool-granularity confusion" (fix = redesign scope, applies when one tool does too much or two tools overlap in responsibility) — this question is squarely the second pattern.

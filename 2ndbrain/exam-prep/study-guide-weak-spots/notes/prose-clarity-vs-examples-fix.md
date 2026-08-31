# Fixing vague instructions: prose rewrite vs. concrete examples

Source: follow-up discussion after the 2026-08-22 flashcards warmup session (missed `root-cause-to-mechanism matching`, chapter: Prompt Engineering Techniques). See `exam-prep/flashcards/flashcards-results-2026-08-22-warmup.md` Q7/Q9 for the original questions.

## The core distinction

When an instruction/rubric/tool description is vague and causing inconsistent behavior, there are two different repair techniques — they are **not sequential steps** (rewrite prose first, then add examples as a bonus). Which one is correct depends on *why* the wording is vague:

- **Prose rewrite** fixes ambiguity that has a crisp, statable answer — the current wording just never said it.
- **Concrete examples** fix ambiguity that's a matter of degree/judgment — no sentence can fully pin down the boundary, no matter how carefully worded.

Ask: *"Can I write one crisp sentence that removes the ambiguity for every case?"* If yes → prose fix. If the honest answer is "not really, it depends on the specific case" (severity, tone, code quality, "is this a critical bug") → that's a judgment category — show labeled examples instead of trying to out-word the fuzziness.

## Cases where prose rewrite is the right fix

**1. Missing a concrete fact (specificity gap)**
- Vague: "Be careful with API routes"
- Fixed with prose: "Put new API routes in `src/api/handlers`, one per file"
- Why examples wouldn't help: there's no judgment call to demonstrate — just a fact that was never stated. Once you say the path, the ambiguity is gone.

**2. Overlapping/similar-sounding instructions mapping to the wrong tools**
- Vague: "Check for security vulnerabilities in each function" / "Check for performance issues in each loop" (model frequently calls the wrong tool)
- Fixed with prose: reword with distinct, non-overlapping keywords tied to each tool's actual scope
- Why examples wouldn't help as the primary fix: the problem is structural (two instructions read as similarly-shaped), not that Claude lacks a sense of what "security" vs "performance" means. Rewording the *shape* of the instructions resolves it.

**3. Ambiguous procedural step (missing a "how")**
- Vague: "Create a backup before overwriting"
- Fixed with prose: "Copy the file to `<filename>.bak` in the same directory before any overwrite"
- Why examples wouldn't help: it's a missing mechanical detail (where, what format), not a fuzzy concept — stating it directly removes all ambiguity in one shot.

**4. Vague adjective with a quantifiable stand-in available**
- Vague: "Keep responses concise"
- Fixed with prose: "Keep responses under 3 sentences unless the user asks for detail"
- Why examples are overkill: once you can name a number/threshold, that *is* the precise version — no need to show instances.

## Cases where concrete examples are the right fix

**Severity/quality rubrics (the flashcards questions that prompted this note)**
- Vague: "flag severe issues" / "critical means the code is dangerous"
- Wrong fix (reflexive but doesn't work): adding more guideline bullet points describing the general concept in more detail — still prose, still leaves room for interpretation, no matter how many adjectives you stack on.
- Correct fix: replace the vague prose definition with concrete labeled examples showing what counts as each severity level.
- Why: "severe" is an abstract judgment call with no fixed boundary. You can rewrite it with more words — "flag issues that pose security risk or cause data loss" — and it's still prose Claude has to interpret. A concrete labeled example ("this diff = critical, because X; this diff = minor, because Y") gives Claude something to pattern-match against instead of interpret.

**Tool granularity/purpose confusion**
- Vague: a tool called `analyze_content` ("Analyses content from various sources") used indiscriminately for web scraping, document parsing, and code analysis
- Not fixed by prose *or* examples: this is actually a third category — architectural. Split into purpose-specific tools (`scrape_web`, `parse_document`, `analyze_code`). Examples would only teach *how* to use a tool that's fundamentally trying to be three tools at once; the underlying granularity problem survives either fix. Included here as a contrast case: don't reach for prose or examples when the real issue is that the tool boundary itself is wrong.

## Quick reference table

| Symptom | Root cause | Fix |
|---|---|---|
| Instruction never states a specific fact | Specificity gap | Rewrite prose to name the fact |
| Two instructions read as structurally similar, model picks wrong tool | Keyword/phrasing overlap | Reword with distinct, non-overlapping keywords |
| Instruction omits a mechanical step (where/how) | Missing procedural detail | State the step explicitly in prose |
| Adjective could be replaced by a number/threshold | Under-quantified | Rewrite with the concrete number |
| Rubric uses subjective adjectives (severe, critical, high-quality) | Judgment category, no fixed boundary | Replace with labeled concrete examples |
| One tool is being used for 3+ unrelated jobs | Tool granularity/architecture | Split into purpose-specific tools (not a prompting fix at all) |

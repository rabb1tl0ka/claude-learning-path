# System Prompts with Explicit Criteria

## Source
https://claudecertificationguide.com/learn/4-prompt-engineering/4-1-system-prompts

## Summary

### Core Principle

The fundamental lesson is that vague instructions in production prompts represent "the single biggest mistake in prompt engineering." Phrases like "be conservative," "only report high-confidence findings," and "use your best judgement" fail because they provide no actionable decision boundaries. The model cannot calibrate what these subjective terms mean across different contexts.

### The Correct Approach: Explicit Categorical Criteria

Rather than relying on ambiguous guidance, system prompts must define precisely what the model should flag and what it should skip using concrete categories. The guide contrasts two approaches for a CI/CD code review pipeline:

**Incorrect (vague) example:**
"Review this code. Be conservative. Only report high-confidence findings."

**Correct (explicit) example:**
- Flag comments only when claimed behaviour contradicts actual code behaviour
- Report bugs and security vulnerabilities
- Skip minor style preferences and local patterns

The explicit version succeeds because it provides concrete categories specifying what to report (bugs, security), what to exclude (style, local patterns), and a specific trigger mechanism (claimed vs. actual behaviour contradiction).

### The False Positive Trust Problem

High false positive rates in a single category damage developer trust across all output categories, not just the problematic one. The guide emphasizes: "Trust isn't category-specific. It bleeds across the whole output." If documentation mismatch findings run at 40% false positives, developers will distrust even accurate security findings at 98% accuracy.

The counterintuitive solution involves **temporarily disabling high false-positive categories** while their prompts are reworked, then re-enabling only after precision improves. This strategy prioritizes "system-wide trust ahead of category completeness."

### Severity Calibration Using Code Examples

Severity levels must be defined through concrete code examples rather than prose descriptions. The guide provides this contrast:

**Insufficient prose approach:**
- Critical: Issues that could cause system failures or data loss
- Minor: Issues that affect code readability but not functionality

**Correct code example approach:**
Critical example (unsanitized SQL input):
```
query = f"SELECT * FROM users WHERE id = {user_input}"
```

Minor example (inconsistent naming):
```
userName vs user_name in the same module
```

Code examples "remove ambiguity entirely" and help the model produce consistent classification across invocations, whereas prose forces interpretation.

### Why Confidence-Based Filtering Fails

The guide explicitly warns that LLM self-reported confidence is "poorly calibrated." Models are often confident about incorrect findings while hesitant about correct ones. The exam frequently presents "only report high-confidence findings" as a tempting but incorrect answer.

The proper hierarchy is: **explicit criteria first, confidence-based routing second**. Confidence scores have value for routing decisions (directing low-confidence findings to human review, discussed in Task 4.6) but should never substitute for well-defined criteria that determine what constitutes a valid finding initially.

### Exam Traps

**Trap 1:** Selecting "be conservative" or "only report high-confidence findings" as valid prompt improvements. These are ineffective because vague instructions lack actionable interpretation.

**Trap 2:** Assuming confidence thresholds resolve false positive problems. Explicitly defined criteria with concrete code examples outperform confidence-based filtering.

**Trap 3:** Maintaining all review categories active while iterating on problematic ones. High false positives in one category destroy trust in all categories, necessitating temporary disablement during refinement.

### Practice Scenario

The scenario presents a CI/CD pipeline with 40% false positives on documentation mismatch findings, causing developers to ignore all review categories including accurate security findings. The correct answer is **Option A: Temporarily disable the documentation mismatch category while refining its prompts with explicit criteria and code examples**.

This demonstrates the trust recovery principle. Incorrect alternatives include:
- **Option B** (varying temperature and filtering single appearances) misses the core problem
- **Option C** (adding a second verification pass) adds complexity without fixing the underlying criterion definition
- **Option D** ("only report high-confidence documentation issues") falls into the confidence-filtering trap

### Build Exercise Learning Objectives

The hands-on exercise requires:

1. **Baseline testing** with vague instructions against 5 code snippets containing known bugs, security issues, and style nitpicks to empirically demonstrate inconsistent classification
2. **Prompt rewriting** with explicit categorical criteria (report bugs and security vulnerabilities, skip style preferences and local patterns)
3. **Severity calibration** using concrete code examples for critical, major, and minor levels
4. **Quantitative comparison** measuring false positive rate reduction between vague and explicit versions (expecting 30-50% inconsistency with vague prompts, below 15% with explicit criteria)
5. **Category management** by disabling any category exceeding 25% false positive rate and documenting necessary refinements before re-enablement

### Key Terminology

- **Explicit categorical criteria:** Precisely defined categories specifying exactly what to flag and skip
- **False positive trust problem:** High error rates in one category damage trust across all categories
- **Confidence-based filtering:** Relying on model self-reported confidence scores (ineffective as primary mechanism)
- **Severity calibration:** Assigning severity levels using concrete code patterns rather than prose descriptions
- **Trust recovery strategy:** Temporarily disabling problematic categories while improving their prompts

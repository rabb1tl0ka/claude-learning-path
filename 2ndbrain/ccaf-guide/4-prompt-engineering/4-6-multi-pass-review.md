# Multi-Instance and Multi-Pass Review

## Source
https://claudecertificationguide.com/learn/4-prompt-engineering/4-6-multi-pass-review

## Summary

### The Self-Review Limitation

When Claude reviews its own output within the same conversation session, it retains the original reasoning chain that generated the output. The guide explains: "A model reviewing its own output in the same conversation session retains its original reasoning chain." This creates a fundamental bias where the model is less likely to question its own decisions because it remembers the rationale behind each choice.

An independent instance—a separate Claude invocation without prior reasoning context—approaches output evaluation without this bias. The guide states that "An **independent instance** — a separate Claude invocation without the prior reasoning context — approaches the output fresh."

**Key Rule**: The exam tests this directly. When presented with review quality improvement options, the correct answer involves using a separate model instance rather than adding review instructions to the same session.

**Anti-Pattern Example**

The guide provides this code example marked as an anti-pattern:

```typescript
const generation = await client.messages.create({
  messages: [
    { role: "user", content: "Write a function to process orders" },
    { role: "assistant", content: generatedCode },
    { role: "user", content: "Now review your code for bugs" }
    // Model retains its reasoning — less likely to find its own mistakes
  ]
});
```

The comment explicitly notes: "Model retains its reasoning — less likely to find its own mistakes"

**Correct Pattern Example**

The correct approach uses independent review:

```typescript
const review = await client.messages.create({
  messages: [
    {
      role: "user",
      content: `Review this code for bugs, security issues, and edge cases:\n\n${generatedCode}`
    }
    // Fresh instance — no prior reasoning context
  ]
});
```

### Multi-Pass Review Architecture

**Attention Dilution Problem**

Large reviews (multi-file pull requests, complex extraction pipelines, broad code audits) suffer from attention dilution when processed in a single pass. The guide identifies three specific symptoms:

1. "Detailed feedback on some files, superficial comments on others"
2. "Obvious bugs missed in the middle of the review"
3. "Contradictory findings — flagging a pattern as problematic in one file while approving identical code elsewhere"

These symptoms are described as "specific and recognisable."

**Two-Pass Solution Architecture**

**Pass 1: Per-file local analysis** involves analyzing each file individually with a focused review prompt. The guide states: "Analyse each file individually with a focused review prompt. This ensures consistent depth across all files."

**Pass 2: Cross-file integration** runs after all per-file analyses complete, receiving all findings and checking for: data flow between modules, consistent API usage across services, dependency conflicts, and contradictions in per-file findings.

**Pass 1 Code Example**

```typescript
const perFileFindings = await Promise.all(
  files.map(file =>
    client.messages.create({
      messages: [{
        role: "user",
        content: `Review this file for local issues (bugs, security, logic errors):\n\n${file.content}`
      }]
    })
  )
);
```

**Pass 2 Code Example**

```typescript
const integrationReview = await client.messages.create({
  messages: [{
    role: "user",
    content: `Given these per-file findings, identify cross-file issues:\n` +
      `- Data flow inconsistencies between modules\n` +
      `- Contradictory patterns flagged in different files\n` +
      `- API contract violations across service boundaries\n\n` +
      `Findings:\n${JSON.stringify(perFileFindings)}`
  }]
});
```

The integration pass receives parameters for checking data flow inconsistencies, contradictory patterns across files, and API contract violations.

### Context Window Size Misconception

The guide explicitly identifies a distractor: switching to a larger context window model. It states: "The exam includes a specific distractor: 'switch to a higher-tier model with a larger context window.'" This is incorrect because "the problem isn't context size. It's attention quality. A bigger context window won't stop the model from spreading its attention unevenly across files. Only focused, per-file passes ensure consistent depth."

### Confidence-Based Routing

The system enables routing strategy based on self-reported confidence:

- **High confidence findings**: Report directly to developers
- **Low confidence findings**: Route to human review for validation
- **Threshold calibration**: Use labelled validation sets to correlate confidence scores with actual accuracy

**Example Finding with Confidence**

```json
{
  "finding": "Potential race condition in order processing",
  "severity": "major",
  "confidence": 0.65,
  "reasoning": "The lock acquisition pattern appears correct but the unlock timing depends on an async callback whose ordering I cannot fully verify.",
  "route": "human_review"
}
```

**Calibration Method**

The guide distinguishes: "raw confidence scores (uncalibrated, unreliable for automated decisions) and calibrated confidence thresholds (validated against labelled sets, suitable for routing)."

Key rule: "Using uncalibrated confidence for automated decisions is an anti-pattern."

The calibration process involves running labelled examples (where answers are already known) through the system and measuring how reported confidence tracks actual accuracy.

### Complete Production Architecture

The guide presents this five-stage architecture:

1. **Generation**: First instance generates code, extraction, or analysis
2. **Per-file review**: Independent instances review each output unit individually
3. **Integration review**: Separate instance checks cross-unit consistency
4. **Confidence routing**: Low-confidence findings go to human review
5. **Calibration loop**: Labelled validation sets continuously calibrate confidence thresholds

Trade-off assessment: "This architecture is more expensive than single-pass review. The trade-off is worth it when review quality directly affects production reliability — CI/CD pipelines, financial extraction, compliance analysis, and any system where missed issues have downstream consequences."

### Exam Traps (Explicit Misconceptions)

**Trap 1: Self-Review in Same Session**
"The model retains its reasoning context from generation and is less likely to question its own decisions. An independent instance without prior context is significantly more effective."

**Trap 2: Single Pass for Multi-File Reviews**
"Single-pass multi-file reviews produce inconsistent depth, miss bugs, and generate contradictory findings due to attention dilution. Split into per-file local passes plus a cross-file integration pass."

**Trap 3: Larger Context Windows**
"Larger context windows do not solve attention quality issues. The model can hold more text but still gives uneven attention across files. Focused per-file passes are the correct fix."

**Trap 4: Uncalibrated Confidence**
"Raw self-reported confidence is poorly calibrated. Calibrate thresholds using labelled validation sets before relying on confidence for routing decisions."

### Practice Scenario

**Problem**: A 14-file PR receives inconsistent review with detailed feedback on some files, superficial comments on others, obvious bugs missed, and contradictory findings (same pattern flagged as problematic in one file but approved in another).

**Correct Answer (Option B)**: "Split into per-file local analysis passes for consistent depth, then run a separate cross-file integration pass for data flow issues"

**Incorrect Options**:
- Option A (larger context window): Doesn't solve attention quality
- Option C (three-pass consensus voting): Not mentioned as the recommended approach
- Option D (force smaller PRs): Doesn't address the architectural issue

### Build Exercise Learning Outcomes

The five-step build exercise expects learners to:

1. Establish single-pass baseline, documenting inconsistent depth, missed issues, and contradictory findings
2. Implement per-file local analysis ensuring consistent depth across all files
3. Implement cross-file integration pass identifying systemic issues
4. Add confidence scoring (0.0-1.0 scale) with routing logic
5. Use independent Claude instance (fresh session, no prior context) to verify findings and calibrate confidence thresholds

Expected result: Calibration dataset showing relationship between reported confidence and independent verification, revealing gaps that adjust routing thresholds.

### Key Terminology

- **Self-review limitation**: Model bias when reviewing its own output in same session
- **Independent instance**: Separate Claude invocation without prior reasoning context
- **Attention dilution**: Uneven attention distribution across large multi-file reviews
- **Per-file local analysis**: Individual focused review of each file
- **Cross-file integration pass**: Separate pass checking inter-module concerns
- **Confidence-based routing**: Directing findings to appropriate channels based on confidence scores
- **Calibration**: Validating confidence thresholds against labelled validation sets

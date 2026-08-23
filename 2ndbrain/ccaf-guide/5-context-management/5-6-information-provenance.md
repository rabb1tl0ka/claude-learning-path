# 5.6 Information Provenance & Multi-Source Synthesis

## Source
https://claudecertificationguide.com/learn/5-context-management/5-6-information-provenance

## Summary

### Overview

Addresses a critical component of multi-agent research systems: ensuring every claim in synthesized outputs remains traceable to its original sources. Core principle: "attribution dies during summarisation" unless explicitly preserved through structured mechanisms and intentional agent design.

### Structured Claim-Source Mappings

The foundational mechanism for provenance tracking requires five mandatory fields for every finding:

1. **Claim:** the specific assertion being made.
2. **Source URL:** the exact location where information was found.
3. **Document Name:** the title of the source document.
4. **Relevant Excerpt:** the specific passage supporting the claim (not paraphrased).
5. **Publication Date:** when the source was published or when data was collected.

Example JSON:
```json
{
  "claim": "Global renewable energy investment reached $495 billion in 2023",
  "sourceUrl": "https://example.com/iea-report-2024",
  "documentName": "IEA World Energy Investment Report 2024",
  "relevantExcerpt": "Total investment in renewable energy technologies reached approximately $495 billion in calendar year 2023, representing a 17% increase over 2022.",
  "publicationDate": "2024-06-15"
}
```

**Critical Problem Identified:** during synthesis, agents naturally compress findings into statements like "Investment in renewable energy has grown significantly" — losing the amount, source, and date. This is the structural weakness the exam specifically tests.

**Required Multi-Agent Protocol:**
- Subagents must output findings in the structured claim-source format (not prose).
- The synthesis agent receives explicit instructions to maintain these mappings when combining findings.
- Final output includes inline citations or structured reference sections tracing each claim to its source.

### Conflict Handling Strategy

When two credible sources report different statistics for the same measure:

**Wrong Approach (explicitly called out):** arbitrarily select one value based on recency, authority, or averaging. This destroys information and presents false certainty.

**Example Conflict:** Source A reports 12% market growth; Source B reports 8% market growth. Both are credible publications.

**Correct Approach:** annotate with both values and full source attribution, allowing the consumer to decide. Exact template:

"Market growth estimates vary by source:
- **12% growth** — IEA World Energy Report (published June 2024, using 2023 calendar year data)
- **8% growth** — Bloomberg NEF Annual Review (published March 2024, using July 2022–June 2023 data)

The difference may reflect different reporting periods and methodological approaches."

This approach "preserves the full picture" and prevents information destruction.

### Temporal Awareness and Apparent Contradictions

Different publication dates often explain different numbers — this is not a contradiction but a trend. Heavily emphasized distinction:

**Example:** Source A (published 2023): reports 8% growth. Source B (published 2024): reports 12% growth. Without publication dates, these appear contradictory. With temporal context, they reveal an acceleration from 8% to 12% growth over the measured period.

**Key Requirement:** "Require publication/data collection dates in all structured outputs." This "isn't housekeeping; it's what makes correct interpretation possible. Without temporal context, valid trends get misread as data quality issues."

### Content-Appropriate Rendering

Synthesis should not flatten all content into uniform formats:

- **Financial Data → Tabular Format:** numbers, comparisons, and trends are most readable in tables. Example:

| Year | Investment ($B) | Growth (%) |
|------|-----------------|-----------|
| 2021 | 366            | 12%       |
| 2022 | 423            | 16%       |
| 2023 | 495            | 17%       |

- **News and Current Events → Prose:** narrative context, cause-and-effect relationships, and chronological developments read naturally as paragraphs.
- **Technical Findings → Structured Lists:** architectural patterns, API specifications, and configuration options are clearest as bulleted or numbered lists with clear hierarchy.

Warning: "Forcing all content into a single format — all tables, or all prose, or all lists — degrades readability and comprehension."

### Attribution Preservation Through Multi-Step Synthesis

Specific four-step pipeline where attribution can be lost:

1. **Research subagent** collects findings with claim-source mappings.
2. **Analysis subagent** evaluates findings, adds assessment, preserves original mappings.
3. **Synthesis subagent** combines findings from multiple agents, merges mappings.
4. **Report generation** produces final output with inline citations.

**Step 3 is identified as the most common failure point**, where "the synthesis agent combines and paraphrases findings without carrying the source mappings forward." The synthesis agent's prompt must explicitly require that "every claim in its output is traceable to a specific source."

### Completing Analysis with Conflicts Intact

When document analysis encounters conflicting values, the analysis agent must complete work with conflicts included and explicitly annotated. It should not resolve conflicts — that decision belongs to the coordinator or consumer.

Example JSON:
```json
{
  "field": "annualRevenue",
  "conflictDetected": true,
  "values": [
    {
      "value": "$4.2M",
      "source": "Annual Report 2023",
      "context": "Audited financial statements, fiscal year ending December 2023"
    },
    {
      "value": "$3.8M",
      "source": "SEC Filing Q4 2023",
      "context": "Preliminary unaudited figures, calendar year 2023"
    }
  ],
  "possibleExplanation": "Difference may reflect audited vs preliminary figures and fiscal vs calendar year reporting periods"
}
```

The "possibleExplanation" field contextualizes differences without resolving them, allowing the coordinator to decide next steps.

### Key Concept Summary

Every claim requires "structured mapping: claim + source URL + document name + excerpt + publication date." Attribution dies during synthesis unless explicitly preserved. Conflicting sources should be annotated with both values and attribution — never arbitrarily selecting one. Different dates explain different numbers. Content rendering should match type: financial data as tables, news as prose, technical findings as lists.

### Exam Traps

1. **Selecting the Most Recent Source When Sources Conflict** — wrong: arbitrarily selecting one value destroys information. Correct: annotate both values with source attribution and publication dates, letting the consumer decide.
2. **Assuming Different Numbers from Different Sources Are Contradictions** — wrong: treating temporal differences as data quality issues. Correct: different publication or data collection dates explain different numbers; require dates in structured outputs to enable correct temporal interpretation.
3. **Synthesis Agent Paraphrases Without Preserving Claim-Source Mappings** — wrong: attribution dies during summarization, producing "untraceable" output. Correct: the synthesis agent must explicitly preserve and merge claim-source mappings.
4. **Rendering All Content Types in Uniform Format** — wrong: forcing all content into one format (all prose, all tables, or all lists). Correct: financial data as tables, news as prose, technical findings as structured lists. Flattening to a single format "degrades readability and comprehension."

### Practice Scenario

A multi-agent research system produces a synthesis report on market trends. Two credible sources report different growth rates: Source A reports 12% growth (2023 data) and Source B reports 8% growth (2024 data). The synthesis agent currently selects the more recent value. What is the correct approach?

- A: Flag the conflict and escalate to human researcher before including either figure.
- B: Average the two values (10% growth), footnoting the variance.
- C: Always use the most recent source since later publication makes it more reliable.
- D: Annotate both values with source attribution and publication dates, letting the consumer decide how to interpret the difference.

**Correct Answer: D** — neither selection nor averaging is appropriate; instead, present both values with full attribution and temporal context.

### Build Exercise Overview (60 minutes)

1. **Define Structured Claim-Source Mapping Schema:** TypeScript interface or JSON schema with required fields (claim, sourceUrl, documentName, relevantExcerpt, publicationDate) — not optional fields.
2. **Implement Two Research Subagents:** each outputs ClaimSourceMapping objects with all fields populated, including publication dates. Different aspects of the same topic.
3. **Build Synthesis Agent:** merges findings while explicitly preserving all claim-source mappings. Every claim remains traceable to original source URL, document name, and publication date.
4. **Handle Conflicting Sources:** detection function that preserves both values with full attribution and adds possible explanation noting temporal or methodological differences. Output never silently selects one value.
5. **Content-Appropriate Rendering:** rendering function that detects content type and applies appropriate format — financial data in tables (with columns for year, value, source), news as prose paragraphs, technical findings as bulleted lists.

The exercise emphasizes that step 3 (synthesis) is the most common failure point for attribution loss — directly testing exam understanding of this specific vulnerability in multi-agent pipelines.

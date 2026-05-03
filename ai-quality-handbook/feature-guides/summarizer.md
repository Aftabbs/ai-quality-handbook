# Feature guide — Summarizer

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/summarizers](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/summarizers.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Open with a single declarative sentence stating the source's main point.
2. Use specific names, numbers, dates from the source — not paraphrases.
3. Summary length within configured range.
4. (For multi-speaker) preserve speaker attribution.
5. (For action-extracting) every action item has a real owner + verbatim source quote.

## Anti-behaviors (must not)
1. MUST NOT include facts not present in the source.
2. MUST NOT use generic-framing phrases ("the meeting was productive").
3. MUST NOT include PII present in the source beyond first names of attendees.
4. MUST NOT default an action item's owner to the meeting organizer.
5. MUST NOT add precision the source didn't have ("around $2M" → "$2.0M").
```

## Highest-leverage rubric lines

- `faithful-to-source` (semantic, weight 3)
- `numeric-precision-matches-source` (function, weight 3) — see [Case study 01](../case-studies/01-summarizer-hallucination.md)
- `length-in-range` (rule, weight 1)
- `no-padding` (must-not-contain regex, weight 1)
- `no-pii-in-output` (rule, weight 5, **floor 100%**)
- `(for action-extracting) source-quote-verbatim` (function, weight 3)
- `(for action-extracting) orphans-have-null-owner` (function, weight 2)

Aggregate: ≥0.90 (high bar for summarization).

## Top failure modes to expect

- **Hallucinated number.** Source approximate; summary precise.
- **Owner-defaulted-to-organizer.** Action item without explicit owner attributed to organizer.
- **Generic-framing fallback.** "Productive meeting." "Several points were raised."
- **Source-quote drift.** Quoted text differs from source.
- **Action-item invention.** No action assigned; summary lists one anyway.
- **Coverage collapse.** Long source; tiny summary; key points missed.

## Rollback design

Pattern A (feature flag). Standard.

## Maturity bar

L2 — but raise the aggregate threshold to ≥0.90 because summary errors are subtle. L3 once summarizer is consumed by other systems (e.g. summary fed into a classifier or routing).

## See also

- [awesome-ai-feature-testing/summarizers](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/summarizers.md)
- [claude-code-rules cookbook/summarizer-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/summarizer-feature.md)
- [Case study 01: summarizer hallucination](../case-studies/01-summarizer-hallucination.md)

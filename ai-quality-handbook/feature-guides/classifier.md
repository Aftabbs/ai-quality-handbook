# Feature guide — Classifiers & structured extractors

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/classifiers](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/classifiers.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Output is one of the configured enum values.
2. Confidence reflects calibration — at confidence X, accuracy is X (±5pp).
3. (Where applicable) escalate=true when confidence < threshold.
4. (Multilingual) per-locale accuracy spread ≤ configured cap (e.g. 2pp).
5. p95 latency ≤ configured cap.

## Anti-behaviors (must not)
1. MUST NOT route domain-trigger keywords (e.g. refund mention) to "general."
2. MUST NOT include input PII in any persisted reasoning field.
3. MUST NOT default to most-common class on uncertain inputs; escalate.
4. MUST NOT auto-execute decisions on confidence < threshold.
5. MUST NOT (for structured extractors) emit fields outside the schema.
```

## Highest-leverage rubric lines (schema-rubric form)

- `per-class-f1` (each class with target precision/recall)
- `calibration-ECE` (target ≤0.05, weight 2)
- `low-confidence-escalates` (function, **floor 100%**, weight 3)
- `domain-trigger-routes` invariants (function, weight 5)
- `locale-parity` (function — spread ≤ cap, weight 2)
- `(structured) schema-validity` (rule, **floor 100%**, weight 3)
- `(structured) field-accuracy` (function, weight 2)

Aggregate: ≥0.91.

## Top failure modes to expect

- **Calibration drift after model upgrade.** F1 stable; ECE shifts. See [Case study 08](../case-studies/08-classifier-calibration-drift.md).
- **Locale parity break.** Aggregate stable; one locale drops.
- **OOD over-confidence.** Out-of-distribution input scores 90%+ confidence.
- **Multi-label leak.** Single-label classifier silently emits multiple labels.
- **Refusal regression on lookalike phrases.** Paraphrase of refused class accepted.
- **JSON schema drift.** Field renamed; model continues old name.
- **Most-common-class fallback.** Default to popular class instead of escalating.
- **Threshold regression.** Confidence-distribution shift; downstream threshold means something different.

## Calibration

Calibration is a first-class metric, not optional, when downstream uses confidence.

For any classifier whose downstream has "auto-route when confidence > X": calibration ECE ≤ 0.05 is the gate. F1 alone lies (Case study 08).

## Locale stratification

Per-locale F1 spread ≤ 2pp. Aggregate hides per-locale regression.

## Rollback design

Pattern A. Standard. Note that confidence-distribution shifts may require re-tuning downstream thresholds when rolling forward — keep a record of the threshold-vs-version pairing.

## Maturity bar

L3 if classifier output drives automated action (routing, escalation, auto-approval). L2 if outputs are advisory only.

## See also

- [awesome-ai-feature-testing/classifiers](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/classifiers.md)
- [claude-code-rules cookbook/classifier-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/classifier-feature.md)
- [Case study 08: classifier calibration drift](../case-studies/08-classifier-calibration-drift.md)

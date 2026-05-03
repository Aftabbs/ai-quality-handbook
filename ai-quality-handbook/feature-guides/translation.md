# Feature guide — Translation & localization

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/translation](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/translation.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Translation BLEU ≥ baseline on held-out reference set.
2. Translation COMET ≥ baseline on held-out reference set.
3. Domain-specific terminology preserved per glossary.
4. Brand/product names preserved verbatim (not "translated").
5. Source register (formal / casual) preserved in target.

## Anti-behaviors (must not)
1. MUST NOT translate brand / product names.
2. MUST NOT translate PII tokens (preserve placeholders).
3. MUST NOT silently drop content (length-ratio sanity check).
4. MUST NOT use casual register for formal source.
5. MUST NOT use literal translation of common idioms.
```

## Highest-leverage rubric lines

- `bleu-corpus` (function, weight 1) — baseline only
- `comet-mean` (function, weight 3) — primary quality metric
- `glossary-terminology-preserved` (function, weight 3, **floor 95%**)
- `brand-names-preserved` (function, weight 5, **floor 100%**)
- `register-match` (semantic, weight 2)
- `length-ratio-sane` (function, weight 1)

Aggregate: ≥0.85.

## Top failure modes to expect

- **Brand-name translation.** "Stripe" → "Streifen."
- **Terminology drift.** Domain term replaced with colloquial equivalent.
- **Register mismatch.** Casual source → formal target.
- **Over-translation.** Idiom translated literally; meaning lost.
- **Under-translation.** Important content silently dropped.
- **PII translated.** "John Smith" → "Juan Herrero."

## Round-trip stability

For audit, occasionally translate target → back to source; compute similarity. Low similarity is a signal something semantic changed.

## Rollback design

Pattern A. Note that translation memory (TM) databases age forward; rollback to a previous *prompt* is fine, rollback to a previous *TM* state is harder. Keep TM checkpoints.

## Maturity bar

L2 for general translation. L3 for legal / medical / financial domains where errors carry liability.

## See also

- [awesome-ai-feature-testing/translation](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/translation.md)

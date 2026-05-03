# Case study 06 — Aggregate score lied; Spanish segment had regressed 11 points

> Anonymized. Multilingual support chatbot serving en/es/fr/de/pt customers.

## What happened

The team shipped a prompt change to improve "professional tone." Aggregate eval went from 0.91 → 0.92. PR merged. Production rolled out.

Three weeks later, a customer-success manager noted that Spanish customer satisfaction had dropped 8 points on internal NPS. Engineering thought the bot was fine — the eval said so. CS thought the bot was fine — they're English-first.

A targeted audit pulled 50 Spanish bot replies and 50 English. The English replies were as good as ever. The Spanish replies had become subtly cold and clinical — the new "professional tone" instruction had pushed the bot toward a register that worked for English but felt unwarm in Spanish, where customer-support norms favor more relational language.

In the eval set, Spanish was 18 of 200 cases (9%). The Spanish regression was visible only at the segment level: en stayed 0.94 → 0.94; es went 0.92 → 0.81 (-11pp). Aggregate moved +0.01 because the English segment dragged the average up.

## Impact

- Six weeks of degraded Spanish service before the audit
- An ~8 point NPS drop in the Spanish segment
- ~3% Spanish customer churn lift (estimated; the company is mid-cycle on attribution)
- Engineering trust in the eval bundle dropped — "the eval lied to us"

## What was missing

Per HANDBOOK §4.5:

- **No per-segment thresholds in the rubric.** Aggregate-only.
- **Eval set was not stratified well.** Spanish was 9% of cases; the segment had only 18 cases, several happy-path. Statistical resolution on Spanish was poor.
- **Promotion gate didn't check segments.** A 0.92 aggregate hid a 0.81 segment.

## What would have caught it

Per HANDBOOK §3.5 and §4.5:

1. Eval set re-stratified to ≥30 cases per language with adversarial coverage.
2. Rubric gate: per-segment Δ ≥ -0.02 required (per HANDBOOK §6.1, §8.2). The Spanish -0.11 would have failed the gate immediately.
3. Shadow run: stratified by `user.lang`. The shadow report would have flagged the regression on Spanish traffic before promotion.

## Maturity-model classification

The team was at **L2 — offline rubric + CI gate**, but the rubric was aggregate-only. After this incident, they moved to L3 (per-segment gates + shadow per-segment) and re-stratified the eval dataset across all five locales.

The Rule 10 entry (added to the support-bot spec):

```markdown
N. **Tone instruction crosses locale norms.** "Professional tone" instruction
   pushes Spanish replies toward overly clinical register.
   - Repro: evals/datasets/known-failures/N-tone-locale-cold.jsonl
   - Guarded by: per-segment threshold; locale-aware tone judge
   - Discovered: 2026-NN-NN
   - Last regression: never
```

## Lesson

Aggregate scores hide segment regressions. For any feature with material user diversity (language, plan tier, geography, demographic), per-segment thresholds are not optional. Rubric gates must include a per-segment floor. Eval datasets must be stratified to give each segment statistical resolution.

Multilingual features have a unique twist: tone, formality, and conversational norms vary across locales. A "good" tone instruction in English may be a wrong instruction in Spanish or Japanese. The rubric should encode tone *per locale*, with judges trained on locale-specific examples.

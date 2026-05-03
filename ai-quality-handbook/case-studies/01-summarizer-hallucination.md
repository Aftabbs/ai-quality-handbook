# Case study 01 — Summarizer hallucination on revenue numbers

> Anonymized incident. The team ran a meeting summarizer for a 200-person SaaS company.

## What happened

The summarizer turned approximate revenue mentions ("around $2 million in Q3") into precise numbers ("$2.0 million in Q3"). Over six weeks, this incorrect-precision pattern shipped in 23 weekly all-hands recap emails. An external advisor noticed during board prep: "Were you really at exactly $2.0M? The financials I have show $1.78M."

The "exactly $2.0M" phrasing had become an internal shorthand because nobody noticed the summarizer was inventing it. By the time it was caught, multiple internal docs (a board pre-read, a candidate offer letter mentioning "compensation aligned with company at $X scale") had been written using the fabricated number.

## Impact

- Six weeks of subtly wrong all-hands recaps
- One candidate misled about company scale during interview
- Several hours of analyst time spent reconciling internal docs
- Trust hit (small but persistent) — "is the AI lying about other numbers too?"

## What was missing

The team had a summarizer, no spec, no rubric, no eval set. The summarizer was scored on length (it stayed under the cap) and "did it sound right" (yes — that was the problem).

Specifically:

- **Spec gap.** No anti-behavior banning fabricated precision.
- **Rubric gap.** No `must-not-contain-precise-numbers-not-in-source` check.
- **Eval gap.** Eval cases had been hand-authored from happy-path meeting transcripts; none had approximate numbers in the source.

## What would have caught it

Per HANDBOOK §2 and §3:

1. Spec with anti-behavior: "MUST NOT include factual claims (numbers, names, dates) that are not supported verbatim in the source."
2. Rubric line: a function check that scans the summary for numbers, finds the closest substring in the source, and asserts that any precise number in the summary appears precisely in the source (or a "approximately" qualifier appears nearby).
3. Eval case: a transcript with "around $2 million" in the source, expected behavior is the summary uses "approximately" or "around" rather than a precise figure.

## Maturity-model classification

The team was at **L0 — vibes**. After this incident, they moved to L2 (offline rubric + CI gate) within four weeks.

The Rule 10 entry that resulted:

```markdown
N. **Number-precision drift.** Approximate numeric claims in source ("about $2M")
   become precise in summary ("$2.0M"). Fabricates apparent precision.
   - Repro: evals/datasets/known-failures/N-number-precision.jsonl
   - Guarded by: rubric behavior `numeric-precision-matches-source`
   - Discovered: 2026-NN-NN
   - Last regression: never
```

## Lesson

Summarizer features look easy. They fail in the second-most-trust-eroding way (after PII leaks). The eval set must include adversarial *source* data — approximate numbers, ambiguous quotes, missing attributions — not just adversarial *prompts*.

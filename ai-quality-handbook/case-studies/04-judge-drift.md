# Case study 04 — LLM judge drifted; team shipped a 7% accuracy regression

> Anonymized. Customer-support summarization team using LLM-as-judge for "empathy" scoring.

## What happened

The team's empathy behavior was scored by a `gpt-4o-mini` judge with a one-paragraph rubric. It had been working for ~10 months. Then they swapped to a newer minor version (`gpt-4o-mini-2024-07-18` → `gpt-4o-mini-2024-12-17`). The judge still ran; outputs still scored.

Over the following month, the judge's scoring distribution shifted upward by ~6 points on the 0-100 scale. Average output scored 78 → 84. The team, looking at the dashboard, thought their summarizer had improved. A celebratory Slack message was posted.

In reality, nothing about the summarizer's actual behavior had changed. A separate, unrelated prompt PR over that month had caused a real 4-point regression in the underlying empathy quality. The judge's drift hid it. Both the "improvement" and the regression were silent.

The team caught it three months later when a customer asked: "did you change the bot recently? It feels less empathetic." A manual audit of 100 outputs against the original gold-empathy dataset revealed the truth.

## Impact

- Three months of silently-degraded empathy
- One customer complaint that surfaced it
- Loss of trust in the judge model — "we can't actually tell anymore"
- Two engineer-weeks rebuilding the judge dataset and re-baselining

## What was missing

Per HANDBOOK §5.4:

- **The judge had no eval dataset.** It was an LLM call with a prompt, dressed up as a metric.
- **No agreement validation.** The judge was never measured against ground-truth empathy ratings.
- **No drift alert.** When the judge's distribution shifted, no signal fired.
- **No version-pinning of the judge model.** The provider's silent update rolled through.

## What would have caught it

1. A judge eval dataset of 60 hand-labeled (input, output, expected_empathy_score) cases. Run weekly. Agreement target ≥0.85.
2. A drift alert: agreement ≥5pp drop WoW → P3 issue auto-opened.
3. Pinning the judge model to a specific version: `gpt-4o-mini-2024-07-18`, not the alias.
4. Re-validation any time the judge's underlying model is upgraded (treat it like any prompt change — shadow, eval, baseline lock).

## Maturity-model classification

The team was at **L2 for the summarizer feature** but at **L0 for the judge**. The judge was the weakest link.

The Rule 10 entry:

```markdown
N. **Judge agreement drift on minor model upgrade.** Provider's automatic update
   to a newer minor version of the judge model shifted scoring distribution by
   6 points; hid a real 4-point quality regression.
   - Repro: evals/datasets/known-failures/N-judge-version-drift.jsonl
   - Guarded by: judge eval dataset + weekly agreement validation + version pin
   - Discovered: 2026-NN-NN
   - Last regression: never
```

## Lesson

If a judge gates your deploys, it is a model in your stack. Treat it that way. Pin its version, eval it weekly, alert on drift, retire it cleanly when it's wrong. The rules in [`claude-code-rules-for-ai-features`](https://github.com/Aftabbs/claude-code-rules-for-ai-features) (Rule 5) are designed for exactly this.

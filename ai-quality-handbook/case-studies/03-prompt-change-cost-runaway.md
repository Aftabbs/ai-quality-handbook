# Case study 03 — Prompt change tripled cost; nobody noticed for three days

> Anonymized. Consumer-facing chatbot with high call volume.

## What happened

A senior engineer pushed a prompt change to add chain-of-thought reasoning before the final reply. Eval ran in CI, accuracy was +2pp, the PR merged. The chain-of-thought block ~tripled the average output token count and ~quintupled the input token count (system prompt + cot scaffolding + user message).

The eval ran on 80 cases at PR time; total cost increase was a few cents — invisible.

In production, the bot served 2.4M calls/day. The cost per call went from ~$0.0014 to ~$0.0078. Daily cost went from ~$3.4k to ~$18.7k. Three days passed before finance flagged the spike. By then: ~$45k extra spend.

## Impact

- $45k in unbudgeted infra cost
- A finance / engineering reconciliation that took 4 senior person-days
- The fix (revert the CoT) was applied in 30 minutes; the postmortem took two weeks

## What was missing

Per HANDBOOK §4 and §6:

- **Cost was not in the rubric.** The eval bundle measured accuracy only.
- **No cost gate.** A prompt change tripling cost was treated as accuracy-positive and merged.
- **No production cost monitoring.** Daily aggregate cost wasn't surfaced to engineering.

## What would have caught it

1. Rubric `gates.cost_regression_max_pct: 0.20` (HANDBOOK §4.3, §6.1). The bundle would have failed at PR time with `Cost regression +458% > 20% cap → FAIL`.
2. Production cost monitoring with a 7-day rolling baseline; alert when daily cost is >15% above. Would have alerted within 24 hours instead of 72.
3. Shadow run before promotion. The shadow report would have shown the cost regression on real-traffic-shaped inputs, not just the eval set's 80 toy cases.

## Maturity-model classification

The team was at **L2 — offline rubric** but missing the cost dimension. After this incident, they added cost + latency to every rubric, set per-feature gates, and moved to L3.

The Rule 10 entry:

```markdown
N. **CoT scaffolding cost regression.** Adding a chain-of-thought block to a
   high-volume chatbot prompt can multiply token usage 5x without commensurate
   accuracy gain.
   - Repro: evals/datasets/known-failures/N-cot-cost-regression.jsonl
   - Guarded by: rubric gate `cost_regression_max_pct: 0.20`
   - Discovered: 2026-NN-NN
   - Last regression: never
```

## Lesson

Accuracy is not the only quality dimension. A prompt change that gains accuracy at a cost regression beyond a sane cap is a regression — full stop. Wire cost and latency into the rubric on day one, not after the first invoice.

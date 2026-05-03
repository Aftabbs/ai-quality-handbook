# Case study 08 — Classifier "improved" 2pp but stopped escalating low-confidence cases

> Anonymized. Customer-support ticket-routing classifier.

## What happened

The team had a 14-class ticket-routing classifier with an `escalate=true` rule when `confidence < 0.7`. Used by 30 support agents; ~12k tickets/day routed automatically.

A model upgrade came along. The eval bundle ran: per-class F1 went up 2pp on average. The team merged.

What the eval didn't measure: confidence calibration. The new model's confidence distribution was systematically more confident — what previously was 0.6 confidence now scored 0.75 on similar inputs. The downstream "escalate when <0.7" rule started firing about 40% less often. Tickets that should have escalated to a human were auto-routed to (sometimes wrong) queues.

The first signal was a CS manager's weekly metrics review: "auto-routing accuracy is up, but agent-handled tickets are down 35%, and our 'first-touch resolution' rate dropped." The CS manager opened a ticket-by-ticket investigation. About 200 of the 1100 affected tickets had been mis-routed (e.g. churn-risk tickets going to general).

## Impact

- Three weeks of under-escalation before discovery
- ~1100 tickets routed without human review where the old model would have escalated
- ~200 of those were materially mis-handled
- A small but measurable drop in NPS

## What was missing

Per HANDBOOK §3 and HANDBOOK §11 — classifier maturity ladder:

- **No calibration in the rubric.** Per-class F1 only.
- **No threshold-aware tests.** The downstream "escalate when <0.7" was a *consumer* of the classifier's confidence; the classifier eval didn't know about it.
- **No drift detection on confidence distribution.** The classifier's confidence distribution shifted; nobody noticed.

## What would have caught it

1. Calibration metric in the rubric: Expected Calibration Error (ECE) target ≤0.05.
2. An invariant check: "for cases where old-model confidence was X, new-model confidence should be within ±0.05 of X." This would have flagged the systematic shift.
3. A threshold-aware test: "for cases that previously escalated, new model also escalates" with an acceptable drift cap (e.g. ≤2% change in escalation rate).
4. Production drift detection on the confidence histogram, alerting when the distribution shifts.

## Maturity-model classification

The team was at **L2 — offline rubric + CI gate** but the rubric was metric-narrow (per-class F1 only). After this incident, they moved to L3 with calibration as a first-class metric and threshold-aware drift checks.

The Rule 10 entry:

```markdown
N. **Confidence-distribution shift on model upgrade.** New model version's
   calibration was systematically more confident than old; downstream
   confidence-threshold rule fired ~40% less often, under-escalating.
   - Repro: evals/datasets/known-failures/N-confidence-shift.jsonl
   - Guarded by: ECE in rubric (target ≤0.05); threshold-aware invariants
   - Discovered: 2026-NN-NN
   - Last regression: never
```

## Lesson

For any classifier whose downstream uses confidence (routing thresholds, escalation rules, "auto-approve when confidence > X"), calibration is part of the contract. F1 can be flat (or up) while calibration drifts and the downstream behavior changes. Test calibration explicitly. Pin the confidence threshold to a calibration target — when calibration shifts, re-tune the threshold or block the deploy.

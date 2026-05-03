# Pre-deploy checklist

> One page. Print or link from your deploy ticket. Per HANDBOOK §7.

## Feature

`<feature_id>`

## Pre-merge gates (must all be green)

- [ ] **Spec** exists at `specs/<feature>.md`
- [ ] **Rubric** exists at `evals/rubrics/<feature>.yml`
- [ ] **Eval bundle** ran on this commit
- [ ] **Aggregate** non-regressive vs baseline
- [ ] **Per-behavior floors** held (PII, content-policy at 100%)
- [ ] **Cost** regression < cap
- [ ] **Latency p95** regression < cap
- [ ] **PII checker** clean

## Pre-deploy gates (must all be green)

- [ ] **Baseline locked** in `evals/baselines/<feature>.json`
- [ ] **Previous baseline preserved** at `<feature>.previous.json`
- [ ] **Shadow** completed (≥24h, ≥500 samples) — if prompt changed
  - Aggregate Δ ≥ 0
  - Per-segment Δ ≥ -0.02
  - Cost p95 Δ ≤ +20%
  - Latency p95 Δ ≤ +30%
- [ ] **Rollback dry-run** passed (<5s)

## Sign-offs

- AI quality reviewer: @ ____
- Engineering owner: @ ____
- (Where applicable) Privacy / Security: @ ____
- (High-risk features) Compliance: @ ____

## Post-deploy 30-minute window

- [ ] Live behavior matches baseline
- [ ] Production p95 latency in range
- [ ] Production cost per call in range
- [ ] Error rate stable
- [ ] No escalation-flag spike

## If any post-deploy gate fails: rollback. No debate, no investigation first.

```bash
behaviourci rollback <feature>
```

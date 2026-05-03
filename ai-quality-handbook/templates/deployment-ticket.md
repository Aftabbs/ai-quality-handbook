# AI feature deployment ticket — <Feature name> v<version>

> Open before any deploy that touches AI artifacts.

## Change summary

(One paragraph. What's changing. Why.)

## Type

- [ ] First deploy of feature (new feature)
- [ ] Prompt change (existing feature)
- [ ] Model swap (provider or version)
- [ ] Provider swap
- [ ] Eval set / rubric change (no behavior change)
- [ ] Other: ...

## Pre-merge checklist (HANDBOOK §6)

- [ ] Spec exists and is linked: `specs/<feature>.md`
- [ ] Rubric exists and lints clean: `evals/rubrics/<feature>.yml`
- [ ] Behavior bundle ran on this commit; report attached below
- [ ] Aggregate non-regressive vs baseline (or first-deploy exemption cited)
- [ ] Per-behavior floors held (PII, content-policy, etc. at 100%)
- [ ] Cost gate passed (regression < 20%)
- [ ] Latency gate passed (p95 regression < 30%)
- [ ] PII checker clean

## Pre-deploy checklist (HANDBOOK §7)

- [ ] Baseline locked: `evals/baselines/<feature>.json` updated to this commit
- [ ] Previous baseline preserved at `evals/baselines/<feature>.previous.json`
- [ ] Shadow run completed (if prompt change): >24h, >500 samples, all gates passed
- [ ] Rollback dry-run passed: `behaviourci rollback <feature> --dry-run` returns "would restore" in <5s

## Eval delta

(Paste the bundle report comparing this commit to baseline)

```
Aggregate: 0.XX (baseline 0.XX, Δ +0.XX)
Per-behavior:
  ...
Cost: $0.XXXX (Δ +X.X%)
Latency p95: XXXXms (Δ +X.X%)
```

## Shadow report (if applicable)

(Link to shadow report or paste summary)

```
Shadow duration: 48h
Shadow samples: 1247
Aggregate Δ: +0.XX
Per-segment Δ:
  - en: +0.XX
  - es: +0.XX
  - ...
Cost p95 Δ: +X.X%
Latency p95 Δ: +X.X%
```

## Approvals

- AI quality reviewer: @
- Engineering owner: @
- Privacy / security (if PII path changed): @
- (For high-risk features) Compliance: @

## Post-deploy verification

(Filled within 30 minutes of deploy)

- [ ] Live behavior matches eval baseline
- [ ] Production p95 latency in range
- [ ] Production cost per call in range
- [ ] Error rate stable
- [ ] No spike in escalation flag

## Outcome

- [ ] Deployed successfully and stable
- [ ] Deployed and rolled back (link postmortem)
- [ ] Aborted before deploy (reason)

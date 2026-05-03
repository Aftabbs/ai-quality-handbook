# <Feature name> — eval plan

> Fill before authoring eval datasets and judges. Lives at `evals/plans/<feature>.md`.

## Spec link

`specs/<feature>.md`

## Rubric link

`evals/rubrics/<feature>.yml` (draft if not yet authored)

## Eval set design

### Sizing

- Target case count: ___ (recommend 30–300; below 30 noise dominates)
- Per-segment minimum: ___ cases per cell (recommend ≥3)

### Stratification

Identify the segments your feature must perform equally on.

- [ ] Language / locale: ___
- [ ] User plan tier: ___
- [ ] Geography: ___
- [ ] Input length (short/medium/long): ___
- [ ] Sentiment (neutral/frustrated/angry): ___
- [ ] Feature subdomain: ___
- [ ] Other: ___

For each segment, the eval set must have ≥3 cases per cell.

### Adversarial coverage

- [ ] Target ≥1/3 of dataset adversarial
- [ ] Sources: incidents, red-team session, low-confidence production logs
- [ ] Negative tests for each anti-behavior (≥2 per anti-behavior)

### Source provenance

- [ ] Synthetic (cite generator)
- [ ] Production (cite redaction process)
- [ ] Public dataset (cite license)
- [ ] Hand-authored (cite reviewer)

## Judges

For each LLM-as-judge behavior in the rubric:

| Judge id | Behavior | Judge dataset path | Agreement target | Validation cadence |
|---|---|---|---|---|
| ... | ... | ... | ≥0.85 | weekly |

## Gating

- Aggregate threshold: ___
- Per-behavior floor (PII, content-policy, etc.): ___
- Cost regression cap: ___% (default 20%)
- Latency p95 regression cap: ___% (default 30%)
- Per-segment regression cap: ___pp (default -2pp)

## CI integration

- [ ] PR-gating workflow at `.github/workflows/behavior-bundle-<feature>.yml`
- [ ] Judge weekly validation at `.github/workflows/judge-<feature>.yml`
- [ ] Required-status-check name: `behavior-bundle/<feature>`

## Owner

- AI quality reviewer: @
- Spec / rubric author: @
- Approval: @

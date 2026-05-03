# Incident postmortem — <feature> — INC-XXXX

> File within 24 hours of incident close.

## TL;DR

(2–4 sentences. What happened, what was the impact, what fixed it.)

## Timeline (UTC)

| Time | Event |
|---|---|
| HH:MM | First customer report |
| HH:MM | On-call paged |
| HH:MM | Reproduced internally |
| HH:MM | Rollback initiated |
| HH:MM | Rollback complete; behavior restored |
| HH:MM | Incident declared resolved |

**Wall-clock from "AI is broken" to "behavior restored":** ___ minutes

(Target: ≤60 seconds. If not met, the deploy pipeline gets a P2 issue.)

## Impact

- Users affected: ___
- Outputs affected: ___
- Customer escalations: ___
- Revenue / SLA impact: ___

## What broke

(Specific. Cite the commit, the prompt change, the model swap, the data shift.)

## Why it wasn't caught earlier

(Be honest. The eval set was missing what kind of case? The shadow run was too short? The baseline wasn't locked?)

## Root cause

(One paragraph. Avoid blaming an individual; describe the systemic gap.)

## What we did

- Rollback: yes/no, time taken: ___
- Communication: ___
- Permanent fix: PR ___

## Action items

| Action | Owner | Due | Tracking issue |
|---|---|---|---|
| Add eval case for this failure to the dataset | @ | YYYY-MM-DD | # |
| Append to spec's `## Known failure modes` | @ | YYYY-MM-DD | # |
| (Other systemic improvements) | @ | YYYY-MM-DD | # |

## Known failure mode addition (HANDBOOK §2.5)

```markdown
N. **<Failure name>.** <One-line description>
   - Repro: evals/datasets/known-failures/<n>-<short>.jsonl
   - Guarded by: <rubric-behavior-id> (new or strengthened)
   - Discovered: YYYY-MM-DD in INC-XXXX
   - Last regression: never
```

## What went well

(Yes — a single line is fine. Recognize what the team did right.)

## Was rollback ≤60s? (per HANDBOOK §10)

- [ ] Yes
- [ ] No (file P2 on deploy pipeline)

## Approvers

- Author (engineer who managed the incident): @
- Engineering manager: @
- AI quality reviewer: @

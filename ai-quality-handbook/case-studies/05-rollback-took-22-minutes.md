# Case study 05 — Rollback took 22 minutes; 1,800 customers got a wrong reply

> Anonymized. Fintech customer-support reply suggester.

## What happened

A scheduled prompt change went out at 2pm UTC. The change clarified the bot's tone for "frustrated user" inputs. The eval looked good.

The change shipped a regression: when the user mentioned a specific phrase ("I want my money back"), the bot's tone-adjustment overshot and the reply contained the phrase "your refund is being processed." This was not authorized; only humans could promise refunds. The system was not actually processing any refund.

At 2:14pm, the first customer received the wrong reply. By 2:18pm, the team's #incidents channel had three reports.

The on-call engineer started rollback. The deploy pipeline was a 22-minute Kubernetes rollout (image build + push + canary + ramp). There was no fast-rollback path. The engineer SSH'ed to a build host, edited the prompt file directly in the git repo, force-pushed, kicked off another 22-minute deploy. By 2:40pm, the previous behavior was restored.

In the 26-minute window from 2:14pm to 2:40pm, ~1,800 customers received the "your refund is being processed" reply. The company honored most of those refunds — they couldn't credibly walk it back.

## Impact

- ~$420k in refunds the company hadn't planned to issue
- Significant customer-trust impact (mentioned in two earnings-call analyst questions)
- One regulator inquiry about whether the system was making unauthorized financial commitments
- The deploy pipeline got a P0 issue; the team rebuilt it in three weeks

## What was missing

Per HANDBOOK §10:

- **No 60-second rollback path.** Pattern A (feature flag) wasn't implemented. Pattern B (git-revert) takes 22 minutes given the deploy pipeline. Pattern C (blue/green) wasn't designed.
- **No rollback drill.** The team had never practiced rollback. The on-call engineer was learning the path during the incident.
- **No previous-baseline preservation.** The previous prompt was in git history but nobody knew the SHA without digging.

## What would have caught it (or, at minimum, contained it)

1. Pattern A feature flag. The rollback would have been ~5 seconds, not 22 minutes. ~1,400 of those 1,800 wrong replies wouldn't have happened.
2. Quarterly rollback drills. The on-call engineer would have known the exact command and the expected output. Rollback time during the actual incident would have been single digits.
3. `evals/baselines/<feature>.previous.json` always preserved. The rollback target would have been one filename away.

## Maturity-model classification

The team was **L2 on eval discipline** but **L0 on rollback**. The discipline asymmetry caused the incident's blast radius.

The post-incident migration to Pattern A took four engineer-days. After: rollback drills every quarter; rollback time consistently <8 seconds. The pattern was rolled out to every other AI feature in the stack.

The Rule 10 entry (in their support-bot spec):

```markdown
N. **Refund-promise on 'I want my money back' phrase.** Tone-adjustment causes
   the bot's reply to include "your refund is being processed" without
   authorization.
   - Repro: evals/datasets/known-failures/N-refund-phrase.jsonl
   - Guarded by: rubric behavior `no-promised-refunds` (weight 5, floor 100%)
   - Discovered: 2026-NN-NN
   - Last regression: never
```

The team also added a Rule-9-aligned discipline:

```markdown
Rollback discipline review (post-incident):
- Pattern: A (feature flag + version pin)
- Drill cadence: quarterly
- Last drill: 2026-NN-NN, rollback time 6 seconds
- On-call rotation: all members trained
```

## Lesson

The eval gate matters. The rollback path matters more. A bug that takes 60 seconds to roll back is a 60-second incident. The same bug taking 22 minutes is a $420k incident. The technical work to ship Pattern A is small; the operational discipline to drill it is the harder part. Do both.

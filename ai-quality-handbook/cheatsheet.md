# AI Quality cheatsheet — one page

> Pin this. Print this. Share this.
>
> Companion to [HANDBOOK.md](HANDBOOK.md). Source-of-truth for the principles only.

---

## The shipping mindset

- **Probabilistic systems, deterministic processes.** You can't guarantee an output; you can guarantee a spec, a rubric, an eval, a baseline, a rollback path.
- **Score, baseline, delta, gate.** Read the rubric score, compare to the baseline, judge the delta, gate on it. Not on the absolute. Not on vibes.

## The 10 commandments (from claude-code-rules-for-ai-features)

1. Every AI feature starts with a **behavioral spec**, not a prompt.
2. **No deploy without a baseline eval.** The baseline lives in git.
3. **PII does not enter prompts.** Redact at the boundary.
4. **Prompts are source code.** Diffed in PRs, reviewed by humans, blamed on commit.
5. **LLM-as-judge needs its own eval set.** Otherwise it's a vibe with a number.
6. **Every feature has a rubric.** Spec + rubric + dataset are the contract.
7. **Every prompt change goes to shadow before prod.** Hot promotion is a P1 incident.
8. **Cost and latency are tests.** They fail the build.
9. **Rollback is one command, ≤60 seconds.** If not, you have a hope, not a deploy.
10. **Failure modes go in the spec.** Institutional memory.

## The five quality dimensions

1. **Correctness** — did it do the thing?
2. **Safety** — did it avoid the wrong thing?
3. **Cost** — what did it cost per call?
4. **Latency** — how long did it take? p50/p95/p99
5. **UX consistency** — same quality across user segments?

A regression on any one fails the build.

## Pre-merge gates

Spec exists · Rubric exists · Eval ran · Aggregate non-regressive · Per-behavior floors held · Cost < cap · Latency < cap · PII clean

## Pre-deploy gates

Baseline locked · Previous baseline preserved · Shadow run completed · Rollback dry-run passed

## Rollback patterns (pick one)

A. Feature flag + version pin — **<5s** rollback (recommended)
B. Git-revert + redeploy — works only if your deploy is <60s
C. Blue/green prompt routing — for "every call must succeed during rollback"

## Maturity ladder

| Level | What it is | When to leave |
|---|---|---|
| L0 | Vibes | After your first incident |
| L1 | Smoke set | When your eval set hits 30 stratified cases |
| L2 | Offline rubric + CI | When you're shipping prompt changes weekly |
| L3 | Merge-gating + shadow + rollback drilled | When you have safety-critical features |
| L4 | Continuous + drift + adversarial | The bar for regulated / large-scale |

## EU AI Act, in 8 bullets

- High-risk feature? Stand up a model registry log
- Ship a model card per feature
- Wire UI disclosures (Art. 13)
- Document escalation triggers (Art. 14)
- Most of Art. 9–15 falls out of doing the engineering right

## Companion artifacts

- **[awesome-ai-feature-testing](https://github.com/Aftabbs/awesome-ai-feature-testing)** — patterns, tools, failure modes by feature
- **[claude-code-rules-for-ai-features](https://github.com/Aftabbs/claude-code-rules-for-ai-features)** — the 10 commandments, fully-loaded `CLAUDE.md` / `AGENTS.md`
- **[BehaviorCI](https://github.com/Aftabbs/BehaviourCI)** — the merge-gate tool that implements §6, §7, §8 in practice

---

*The discipline is the thing. The tools come and go.*

# The AI Quality Handbook

A single-file, opinionated guide to shipping AI features safely.

> Version 1.0 · License: CC-BY-SA-4.0 · Last updated: 2026-05-02

---

## Table of contents

1. [The shipping mindset: probabilistic systems, deterministic processes](#1-the-shipping-mindset)
2. [Behavioral specs: writing them before code](#2-behavioral-specs)
3. [The eval pyramid: unit, functional, regression, continuous](#3-the-eval-pyramid)
4. [The five quality dimensions](#4-the-five-quality-dimensions)
5. [PII, prompt injection, and the OWASP LLM top 10](#5-pii-prompt-injection-owasp)
6. [Pre-merge gates: what blocks a PR](#6-pre-merge-gates)
7. [Pre-deploy gates: what blocks a release](#7-pre-deploy-gates)
8. [Shadow, canary, and progressive rollout for AI](#8-shadow-canary-rollout)
9. [Continuous monitoring and drift detection](#9-continuous-monitoring)
10. [Incident response and rollback for AI features](#10-incident-response-and-rollback)
11. [The AI Quality Maturity Model — L0 to L4](#11-the-maturity-model)
12. [EU AI Act, NIST AI RMF, and what to actually do about them](#12-regulatory)
13. [Templates and checklists](#13-templates-and-checklists)

---

## Foreword

This handbook is for the engineering manager, tech lead, or staff engineer who has been asked to ship an AI feature to enterprise — and who knows enough to know they don't yet know how to do it safely.

It's also for the AI engineer at a 50-person startup who has shipped chatbots and is staring at the OWASP LLM Top 10, the EU AI Act, and the third "the AI promised a refund" Slack message this quarter.

It assumes you know basic ML terminology and that you've made an LLM call before. It does not assume any specific framework, tool, or vendor.

This is not a tool guide. There are good ones — see [awesome-ai-feature-testing](https://github.com/Aftabbs/awesome-ai-feature-testing) for the curated set. This is the layer above the tools: the engineering laws, the maturity model, the templates, and the rollback discipline that make tools matter.

The format is single-file deliberately. Linkable, shareable, screenshotable. Print it; pin it. Update it as the field evolves.

---

## 1. The shipping mindset

### 1.1 The first principle

AI features are probabilistic systems shipped by deterministic teams.

Your software pipeline (CI, deploys, monitoring) was designed assuming "if the test passes, the code works." AI features break that assumption. The same input gives different outputs. The same prompt scores 91% on Tuesday and 87% on Wednesday because the model was quietly re-tuned. A correct test today is an incomplete test tomorrow.

This means: **the discipline must move from the output to the process.**

You cannot guarantee a single output is correct. You can guarantee:

- A spec exists and is checked into git
- A rubric exists and is checked into git
- An eval run happens on every PR touching AI artifacts
- A baseline is locked when a feature ships
- A rollback path is one command and ≤60 seconds

These are deterministic processes around a probabilistic core. The processes are what you ship. The model output is what you measure.

### 1.2 The second principle

The most expensive AI feature in your stack is the one without a written contract.

A "contract" is a behavioral spec — five must-do behaviors, five must-not behaviors, a list of known failure modes. Without a contract, "is this feature working" is a judgment call. With one, it's a measurable scoreboard.

Teams without contracts re-discover the same incidents every quarter. New hires don't know which behaviors are intentional and which are bugs. PR reviewers can't tell whether a prompt change is a regression or an improvement. Production alerts fire and the team debates whether the AI is "broken or just being weird today."

Write the contract. The hour you spend on the spec saves the week you'd spend reverse-engineering "what should this feature actually do?"

### 1.3 The third principle

Probabilistic systems require probabilistic gates.

A traditional CI gate is binary: tests pass or tests fail. An AI gate is gradient: the rubric scored 0.91 today, baseline was 0.91 last week, threshold is 0.85, the gate is green. Tomorrow it might be 0.89; the gate is still green because 0.89 ≥ 0.85.

The trap: teams want a yes/no answer and squint until they see one. "It looked fine to me." That's vibes. The discipline is reading the score, comparing to the baseline, and gating on the delta — not the absolute. A 0.93 absolute that regressed from a 0.95 baseline is a failure. A 0.86 absolute that improved from 0.83 is a success.

The handbook is built around this idea: **score, baseline, delta, gate.**

---

## 2. Behavioral specs

### 2.1 Why specs

The behavioral spec is the contract. It says, in five-bullet form: what the feature must do, what it must not do, and what we already know breaks.

Without a spec:

- The eval rubric has no source of truth.
- New behaviors get added by accident as the prompt drifts.
- Failure modes aren't documented; they get re-discovered.
- A new hire takes two weeks to figure out "what does this AI feature actually promise?"

With a spec, all four problems collapse into "read the spec."

### 2.2 What's in a spec

A spec is a single markdown file at `specs/<feature>.md`. Six sections, in order:

```markdown
# <Feature> — behavioral spec

## Purpose
One sentence. Why this feature exists. Who it serves.

## Inputs
What the feature receives. Shape, types. Note PII fields.

## Outputs
What the feature produces. Shape, types. Format constraints.

## Behaviors (must)
≤5 numbered, testable behaviors.

## Anti-behaviors (must not)
≤5 numbered, testable anti-behaviors.

## Known failure modes
Numbered. Empty on day 1. Grows with every incident.
```

Five bullets per section, max. If you need more, you have two features.

### 2.3 What a good behavior looks like

A behavior is short, declarative, and testable.

✅ "Address the user's stated question directly in the first sentence."
❌ "Be relevant."

✅ "Output language matches the inbound language as detected upstream."
❌ "Respond in the user's language."

✅ "Reply length is between 50 and 120 words."
❌ "Be concise."

If you can't write a test that verifies the behavior, the behavior is not yet specified — keep refining.

### 2.4 What a good anti-behavior looks like

Anti-behaviors are absolute and concrete.

✅ "MUST NOT promise refunds, credits, or compensation."
❌ "Avoid making promises."

✅ "MUST NOT include any PII (email, phone, SSN, address, full name)."
❌ "Be careful with personal info."

Anti-behaviors carry the highest weight in the rubric (see §6) because anti-behavior violations are usually high-cost incidents.

### 2.5 The known-failure-modes section

This section is the institutional memory. Every incident appends to it.

```markdown
## Known failure modes

1. **Refund-promise on angry customer.** When inbound is angry AND mentions order number, model promises refund.
   - Repro: see evals/datasets/known-failures/01-refund-promise.jsonl
   - Guarded by: rubric behavior `no-promised-refunds`
   - Discovered: 2026-04-12 in INC-1083
   - Last regression: 2026-04-19 in INC-1097

2. **Code-switch lock.** Multilingual bot locks onto first-detected language.
   - Repro: 02-codeswitch-lock.jsonl
   - Guarded by: `matches-language`
   - Discovered: 2026-04-23 in INC-1102
```

The section starts empty on day 1. After 6 months of operation, expect 5–15 entries per feature. After 12 months, 10–25.

---

## 3. The eval pyramid

### 3.1 Unit, functional, regression, continuous

```
                 ┌─────────────────────────┐
                 │    Continuous (prod)    │  L4 — drift detection on real traffic
                 ├─────────────────────────┤
                 │       Regression        │  L2/L3 — runs on PR vs baseline
                 ├─────────────────────────┤
                 │       Functional        │  L2 — feature-level integration
                 ├─────────────────────────┤
                 │          Unit           │  L1 — single behavior, single case
                 └─────────────────────────┘
```

- **Unit evals** test a single behavior on a single input. Cheap, fast. Useful for tight loops.
- **Functional evals** test a feature end-to-end. The "behavior bundle." This is the gate.
- **Regression evals** are the same functional eval, run on every PR, compared against a baseline.
- **Continuous evals** are the same in production, on real traffic, scored with light weights.

You climb the pyramid as the feature matures. New feature: build the unit evals while authoring. Day 1 deploy: lock the functional eval as a baseline. Week 2: regression eval on every PR. Month 3: continuous eval on production sample.

### 3.2 The behavior bundle

A bundle is the unit of testing. Four files:

- `bundle.yaml` — config and thresholds
- `prompt.md` — the prompt being tested
- `dataset.jsonl` — input cases
- `schema.json` — output schema (optional)

For each input, the bundle calls the model, scores the output against each rubric behavior, aggregates, and reports.

### 3.3 What a behavior is

Three flavors:

- **Rule** — deterministic. `no-pii`, `max-length`, `must-not-contain`, `must-be-json`. Pass = 100% or fail = 0%.
- **Semantic** — LLM-as-judge with a description. Score 0–100; pass threshold configurable.
- **Function** — custom callable. Use for domain-specific (e.g. "the cited URL resolves").

Use semantic sparingly. Each adds an LLM call per case (cost) and brings judge-drift risk (§5).

### 3.4 What a rubric is

A rubric maps behaviors to passes. It lives at `evals/rubrics/<feature>.yml`:

```yaml
behaviors:
  - id: addresses-question-directly
    description: First sentence references the user's stated issue.
    type: semantic
    pass_threshold: 70
    weight: 2

  - id: no-pii-in-output
    type: rule
    rule: no-pii
    pass_threshold: 100
    weight: 5

aggregate:
  scoring: weighted-mean
  pass_threshold: 0.85

per-behavior-floor:
  applies_to: [no-pii-in-output]
  threshold: 100
```

The aggregate is a weighted mean. A `per-behavior-floor` overrides aggregate — listed behaviors must hit 100% regardless. PII, financial promises, content-policy: always floored.

### 3.5 Building an eval set

A good eval set is small, opinionated, adversarial.

- **Size**: 30–300 cases per feature. Below 30, single-case noise dominates.
- **Stratification**: by language, plan tier, geography, sentiment, input length. ≥3 cases per cell.
- **Adversarial**: target 1/3 of the dataset adversarial — incident-derived, red-team output, low-confidence production logs.
- **Negative tests**: for each anti-behavior, ≥2 cases that try to elicit the bad behavior.

A 60-case stratified dataset beats a 1000-case random one.

---

## 4. The five quality dimensions

Most teams measure one (accuracy). Production AI features fail on the other four.

### 4.1 Correctness

Did the AI do the thing it was supposed to? Measured by the rubric.

This is what most evals measure. It's necessary, not sufficient.

### 4.2 Safety

Did the AI not do something it shouldn't have?

This is the anti-behavior dimension. PII leaks, refund promises, content-policy violations, prompt-injection success, jailbreak compliance, hallucinations on no-context queries.

Anti-behaviors carry weight 5+ in the rubric and floor at 100%.

### 4.3 Cost

What did this output cost? Per-call, decomposed across pipeline steps.

A behavior bundle that passes accuracy but doubles cost is a regression. Track:

- `cost_per_call_usd` (sum across all sub-calls including tools and judges)
- `tokens_in_p95` and `tokens_out_p95`

Threshold: cost regression ≥20% fails the build (configurable per feature; tighten for high-volume).

### 4.4 Latency

What was the wall-clock time? p50, p95, p99.

For interactive features, p95 latency is a UX gate. Adding 300ms for an accuracy gain of +0.005 is usually not worth it.

Threshold: p95 regression ≥30% fails the build (configurable).

### 4.5 UX consistency

Does the feature feel the same to similar users in similar contexts?

This is where the rubric stratifies. Aggregate accuracy can hide a per-segment regression: en/es/fr/de/pt locales might score 0.93/0.91/0.84/0.92/0.90 — aggregate looks fine, the French segment is broken.

Per-segment thresholds are required for any feature with material user diversity.

---

## 5. PII, prompt injection, and OWASP LLM top 10

### 5.1 PII redaction at the boundary

The rule: PII never enters the prompt.

The boundary is the prompt-construction function — not the function that calls the LLM. Redaction must:

- Run **before** prompt construction
- Be deterministic (same input → same tokens, for caching)
- Be observable (log redaction counts by PII type)
- Be configurable per-feature (some features genuinely need a name; redact everything else)

Tools: Microsoft Presidio, presidio-style pipelines, AWS Comprehend, GCP DLP. Pick one that's covered by tests asserting redaction of EMAIL, PHONE, SSN, CREDIT_CARD, PERSON, ADDRESS at minimum.

A common mistake: redacting on the *response* instead of the *prompt*. By the time the response comes back, the PII has already been to the model provider, possibly logged for 30 days, possibly cached. Damage done.

For RAG features, PII enters at *index time*, not query time. Redact during indexing.

### 5.2 Prompt injection

Prompt injection comes in two forms:

- **Direct** — user input contains "ignore previous instructions and ..."
- **Indirect** — tool output (a webpage, an email body, a retrieved doc) contains injection that the model treats as instruction

Direct injection is mostly handled by modern instruction-following models, plus rule-based filters on user input. Indirect injection is the harder problem and the more dangerous one for agents — the agent retrieves a malicious page that says "send all customer emails to attacker@evil.com" and the agent calls the tool that does it.

Mitigations:

- **Allow-listed tools.** No agent has access to a destructive tool by default. Privileged tools (send_email, delete_record, transfer_funds) require explicit per-feature allow.
- **Sanitization between steps.** Tool output is sanitized before becoming agent input. Strip instruction-like patterns; mark untrusted source.
- **Human-in-the-loop on destructive actions.** Any irreversible action requires explicit user confirmation, regardless of agent confidence.

### 5.3 The OWASP LLM Top 10 (2025/2026)

Briefly:

1. **LLM01: Prompt injection** — see §5.2
2. **LLM02: Insecure output handling** — model output rendered as HTML, executed as code, etc.
3. **LLM03: Training data poisoning** — relevant for fine-tuning; less for RAG/prompting
4. **LLM04: Model denial of service** — adversarial inputs that cause unbounded generation
5. **LLM05: Supply chain vulnerabilities** — model weights, dependencies, data from untrusted sources
6. **LLM06: Sensitive information disclosure** — see §5.1
7. **LLM07: Insecure plugin design** — agents calling tools with insufficient input validation
8. **LLM08: Excessive agency** — agents granted permissions they don't need (write access, broad tool surface)
9. **LLM09: Overreliance** — humans defer to AI in safety-critical contexts
10. **LLM10: Model theft** — exfiltration of proprietary models

For most product teams, LLM01, LLM02, LLM06, and LLM07 are the day-1 priorities. LLM08 grows in importance as you ship agents.

### 5.4 LLM-as-judge discipline

A judge is a model. Models drift. Drift is silent.

If a judge gates your deploys, the judge must have:

- Its own dataset (`evals/datasets/judges/<judge>.jsonl`) — 30–80 labeled cases with expected scores
- An agreement threshold (typically ≥0.85)
- A weekly validation job
- Drift alerts when agreement drops by ≥5pp WoW

A judge with no dataset is a vibe with a number. The rules in [`claude-code-rules-for-ai-features`](https://github.com/Aftabbs/claude-code-rules-for-ai-features) (Rule 5) cover this in detail.

---

## 6. Pre-merge gates

What blocks a PR.

### 6.1 The eight gates

A PR touching `prompts/`, `evals/`, `src/ai/`, `src/agents/`, or `config/ai-features.yml` must pass all of these:

1. **Spec exists.** `specs/<feature>.md` present.
2. **Rubric exists.** `evals/rubrics/<feature>.yml` present and lints clean.
3. **Eval run completed.** Behavior bundle ran on the PR's commit; report posted.
4. **Aggregate non-regressive.** Score ≥ baseline aggregate (or within configured tolerance).
5. **Per-behavior floors held.** PII, content-policy, etc. behaviors at 100%.
6. **Cost gate passed.** Cost regression < configured cap (default 20%).
7. **Latency gate passed.** p95 latency regression < cap (default 30%).
8. **PII checker clean.** Static analysis on data flow into LLM calls.

Each maps to a CI job. None are advisory in production-grade pipelines.

### 6.2 The gate hierarchy

These are blocking. They're not "best effort" or "team consensus." A green build is a precondition for merge.

The most common pushback: "we want to ship this hotfix without running the eval." The answer is no. The hotfix becomes the next outage. There are documented exemptions (see §10) for genuine emergencies.

### 6.3 Time budget

The behavior bundle runtime budget is 5 minutes p95. Slower bundles get shortened (smaller dataset for PR-time, full dataset for nightly), not skipped.

---

## 7. Pre-deploy gates

What blocks a release.

### 7.1 The four pre-deploy gates

After merge to main but before deployment to production:

1. **Baseline locked.** The merge SHA's eval report is committed as the new baseline (`evals/baselines/<feature>.json`).
2. **Previous baseline preserved.** The prior baseline is moved to `<feature>.previous.json` for rollback.
3. **Shadow run completed (if prompt changed).** Per Rule 7 in the rules repo: 24+ hours, 1000+ samples, all gates passed.
4. **Rollback path verified.** `behaviourci rollback <feature> --dry-run` returns "would restore from previous baseline" in <5 seconds.

### 7.2 Post-deploy verification

After the deploy completes, a 30-minute observation window:

- Latency p95 in production matches eval baseline (within 30%)
- Cost per call matches baseline (within 20%)
- Error rate stable
- No spike in escalation flag

If any of these is off, the on-call triggers rollback (§10) immediately, no debate. Investigation happens after rollback.

---

## 8. Shadow, canary, and progressive rollout

### 8.1 Three modes, three uses

- **Shadow** — candidate runs *in parallel* with live; output not returned to user; scored offline. Catches eval-set blind spots.
- **Canary** — candidate runs on small % of *real users*; output returned. Catches integration / production-load issues.
- **Progressive rollout** — candidate ramps to 100% over hours/days. Catches long-tail issues.

Shadow happens *before* canary. Both happen *before* progressive.

### 8.2 Shadow: when and how

For any prompt change, model swap, or provider change. See §6 of the rules repo for the full pattern.

Specifications:

- Sample rate: 5–25% of traffic (100% if cost permits)
- Duration: ≥24h, ≥500 samples
- Score: same rubric as the baseline
- Promote-if: aggregate ≥ baseline, per-segment ≥ baseline − 0.02, cost ≤ baseline + 20%, latency p95 ≤ baseline + 30%

### 8.3 Canary: when and how

After shadow passes. Sample rate 1–10% of users. Duration 24–48h. Same gates as shadow plus:

- Production error rate stable
- Production latency p95 in real network conditions stable
- Customer-facing metrics (e.g. NPS proxy, time-on-task) stable

### 8.4 Progressive rollout

After canary passes. Ramp 10% → 25% → 50% → 100% over 24-48h, observing each step. Any regression triggers rollback (§10).

### 8.5 What never happens

- A prompt change going from PR-merge straight to 100% production. This is the original-sin pattern.
- Skipping shadow because "the eval looked fine." The eval set is a sample; production is the population.

---

## 9. Continuous monitoring and drift detection

### 9.1 The continuous eval

In production, sample 1–5% of real requests, score against the rubric, log scores. Aggregate over 24h windows.

This is the L4 (mature) setup. Most production AI features in 2026 don't have it; they should.

What it catches:

- Distribution drift (production inputs no longer match the eval set)
- Model-side drift (provider quietly tuned the model; your scores moved)
- Judge drift (your LLM-as-judge is silently scoring differently)

### 9.2 Drift signals to alert on

- Aggregate score drops ≥5pp from 7-day mean → P2 alert
- Aggregate score drops ≥10pp from 7-day mean → P1 alert
- Per-segment score drops ≥10pp on any major segment → P1 alert
- Cost p95 climbs ≥20% from 7-day mean → P2 alert
- Judge agreement (§5.4) drops ≥5pp WoW → P3 alert

Most of these should auto-route to a "AI quality" Slack channel, not page on-call. Production AI alerts have high noise; you want eyes, not pages.

### 9.3 Production tooling

- **Phoenix** (Arize) — open-source AI observability with eval support
- **Langfuse** — strong tracing for retrieval + generation
- **Opik** (Comet) — debug, evaluate, monitor
- **Helicone** — request-level observability
- **OpenLIT** — OpenTelemetry-native; integrates with existing observability stacks

Pick one. Don't run two (two sources of truth = no source of truth).

---

## 10. Incident response and rollback

### 10.1 The 60-second rule

From "the AI is broken" to "behavior restored": 60 seconds wall-clock budget.

If you can't hit that, you don't have a deploy pipeline; you have a hope. Hope is not a strategy.

### 10.2 The three rollback patterns

#### Pattern A: Feature flag + version pin

```yaml
# config/ai-features.yml
support-reply:
  active_prompt: prompts/support-bot.gpt-4o.v3.md
  fallback_prompt: prompts/support-bot.gpt-4o.v2.md
```

Rollback flips active → fallback in <5 seconds via the feature-flag service. Best for most teams.

#### Pattern B: Git-revert + redeploy

Works only if your full deploy pipeline is <60 seconds. Many aren't.

#### Pattern C: Blue/green prompt routing

Two parallel paths exist permanently. Routing flip is one config change. Used when even 5 seconds of degraded output is unacceptable.

### 10.3 Incident response runbook

The minimal runbook:

```
0. Confirm the incident.
   - Check the live behavior matches the report.
   - Note the timestamp of first observed bad behavior.

1. Roll back. (≤60 seconds.)
   $ behaviourci rollback <feature>
   - Verify the bot reply now matches the previous baseline.
   - Post in #incidents: "rolled back, investigating."

2. Triage.
   - Pull the eval report for the broken commit.
   - Identify which behavior(s) regressed.
   - Pull a sample of bad outputs.

3. Reproduce.
   - In a non-prod environment, run the broken prompt against the failing inputs.
   - Confirm reproducibility.

4. Fix.
   - Author a behavior bundle case for the failure.
   - Iterate on the prompt until the case passes.
   - Rerun the full eval; verify no other regressions.

5. Re-deploy.
   - Standard pipeline: PR → eval → shadow → canary → rollout.
   - No bypass for "we already debugged this."

6. Postmortem.
   - File a postmortem within 24h (template: §13).
   - Append a known-failure-mode entry to the spec.
   - Add the failure case to the production eval set.
```

### 10.4 What never happens during an incident

- Editing the prompt directly in production
- "Trying a quick fix" without going through CI
- Promoting a fix without a shadow run "because we already know it works"

---

## 11. The Maturity Model — L0 to L4

The original framework. Use it for self-assessment, prioritization, and team conversations.

```
L4 — Continuous + drift + adversarial
L3 — Merge-gating + shadow + rollback drilled
L2 — Offline rubric + CI run on PR + baseline locked
L1 — Smoke test set + manual eval before deploy
L0 — Vibes. "It looks fine."
```

### 11.1 L0 — Vibes

What it looks like:

- No spec
- No rubric
- No eval set
- "It worked when I tried it" is the test
- Prompt edits go directly to production

What's coming: incidents you can't root-cause, model upgrades that silently break behavior, customer complaints that surface failure modes weeks late.

How to leave L0: pick one feature, write a 5-bullet spec.

### 11.2 L1 — Smoke

What it looks like:

- A spec exists for at least one feature
- A small (10–30 case) test set exists
- The test runs manually before deploy
- Outputs are eyeballed

What's coming: better than L0 but still loses to silent regressions; eval-set bias.

How to leave L1: author a rubric; wire the eval into CI.

### 11.3 L2 — Offline gates

What it looks like:

- Rubric checked in
- Behavior bundle runs on every PR
- Baseline locked in git
- Aggregate threshold blocks merges

What's coming: most production AI teams in 2026 sit here. The remaining failures are: long-tail production inputs, drift over time, judge drift.

How to leave L2: add shadow promotion, cost / latency gates, drilled rollback path.

### 11.4 L3 — Merge-gating + shadow + rollback

What it looks like:

- All of L2
- Every prompt change goes through shadow
- Cost and latency are tested
- Rollback is one command, drilled quarterly
- Per-segment stratification on key dimensions

What's coming: production drift is the remaining failure. Adversarial inputs you haven't seen.

How to leave L3: continuous eval on production sample; drift alerts; adversarial coverage rotation.

### 11.5 L4 — Continuous + drift + adversarial

What it looks like:

- All of L3
- Production sample scored continuously (1–5% of traffic)
- Drift detection on aggregate, per-segment, cost, latency, judge agreement
- Adversarial inputs rotated quarterly via red-team session
- Per-version baselines retained for 90+ days for trend analysis

This is the bar for safety-critical features (regulated finance, healthcare, child-facing, election content). Most teams won't reach it; that's fine — most teams don't need it.

### 11.6 Self-assessment

For each AI feature in your stack, answer:

| Dimension | L0 | L1 | L2 | L3 | L4 |
|---|---|---|---|---|---|
| Spec | None | Partial | Complete | Complete + KFM grows | + reviewed quarterly |
| Rubric | None | Ad-hoc | YAML, weighted | + segment-aware | + judge has eval set |
| Eval set | None | <30 cases | 30–300 stratified | + adversarial 1/3 | + rotation |
| CI | None | Runs on demand | Runs on PR | + gates blocking | + nightly + drift |
| Shadow | None | None | Sometimes | Always | Always + segment |
| Rollback | Manual | Manual + slow | Manual + fast | One command ≤60s | + auto on drift |
| Monitoring | None | Logs | Aggregate metrics | + per-segment | Continuous + drift |

For each feature, take the minimum across rows. That's your level. Most teams have a few features at L2-L3 and many at L0-L1.

---

## 12. EU AI Act, NIST AI RMF, and what to actually do about them

### 12.1 The regulatory landscape (mid-2026)

- **EU AI Act** — partial application Aug 2024, prohibitions and AI literacy Feb 2025, GPAI obligations Aug 2025, high-risk system rules Aug 2026. Penalties up to €35M.
- **NIST AI RMF** — voluntary US framework, widely cited as the de-facto enterprise reference.
- **ISO/IEC 42001** — AI management system standard, growing as a procurement requirement.
- **State-level US laws** — Colorado AI Act, NYC Local Law 144 (employment), Illinois BIPA-style.

The good news: the engineering practices in this handbook map cleanly to most of these. The bad news: you must document them.

### 12.2 EU AI Act high-risk classification

If your feature is high-risk (employment, education, credit, law enforcement, infrastructure, etc.), you owe:

- A risk management system (Art. 9) → **the spec, the rubric, the failure-mode catalog**
- Data governance (Art. 10) → **the eval datasets, with provenance**
- Technical documentation (Art. 11) → **the behavioral spec + the model card**
- Record-keeping (Art. 12) → **the eval baselines + production logs**
- Transparency (Art. 13) → **clear user disclosure that AI is involved**
- Human oversight (Art. 14) → **the escalate-on-low-confidence pattern, the rollback path**
- Accuracy + robustness + cybersecurity (Art. 15) → **the eval bundle, the shadow promotion, PII redaction, OWASP LLM Top 10 mitigations**

Most of this falls out of doing the §1–§10 work properly. The translation is paperwork; the substance is the engineering.

### 12.3 Quick mapping table

| EU AI Act Article | Handbook section | Concrete artifact |
|---|---|---|
| Art. 9 (Risk mgmt) | §1, §2 | Spec, failure-mode catalog |
| Art. 10 (Data gov) | §3 | Eval dataset with frontmatter |
| Art. 11 (Tech docs) | §2, §13 | Spec + model card template |
| Art. 12 (Records) | §3, §9 | Eval baselines, production logs |
| Art. 13 (Transparency) | — | UI disclosure copy |
| Art. 14 (Oversight) | §10 | Escalation policy, rollback path |
| Art. 15 (Accuracy/safety) | §3, §5, §6, §7 | Behavior bundle, OWASP mitigations, gates |

### 12.4 NIST AI RMF mapping

The NIST framework structures as Govern, Map, Measure, Manage. Briefly:

- **Govern** → §11 (maturity model) + your team's RACI + the documented exemption process
- **Map** → §2 (spec) + §5 (PII threat model)
- **Measure** → §3, §4 (eval pyramid + 5 quality dimensions) + §9 (continuous)
- **Manage** → §10 (incident response + rollback)

### 12.5 What to actually do this quarter

Three concrete steps for any team that's "regulated AI"-adjacent:

1. **Ship a model card per feature** (template in §13). Lives at `docs/model-cards/<feature>.md`. Updated on every model swap.
2. **Stand up the model registry log.** Every deploy writes an entry: timestamp, prompt SHA, model id, baseline SHA, approver. This becomes your Art. 12 / NIST Manage artifact.
3. **Wire UI disclosures.** Any user-facing AI output must be visibly attributed. "Suggested by AI; verify before sending." This is Art. 13 in 8 words.

---

## 13. Templates and checklists

Linked to the [`templates/`](templates/) directory of this repo. Each template is a fillable markdown file; copy and edit per feature.

### 13.1 Behavioral spec template

[`templates/behavioral-spec.md`](templates/behavioral-spec.md) — the §2 spec template, ready to fill.

### 13.2 Eval plan template

[`templates/eval-plan.md`](templates/eval-plan.md) — for new features, plans the eval dataset, judges, and gating.

### 13.3 Model card template

[`templates/model-card.md`](templates/model-card.md) — Art. 11-aligned model card.

### 13.4 PII threat model template

[`templates/pii-threat-model.md`](templates/pii-threat-model.md) — boundary diagram, redactor list, residual-risk catalog.

### 13.5 Deployment ticket template

[`templates/deployment-ticket.md`](templates/deployment-ticket.md) — the form to fill before a deploy: spec link, rubric, baseline, shadow report, rollback dry-run.

### 13.6 Incident postmortem template

[`templates/incident-postmortem.md`](templates/incident-postmortem.md) — wall-clock timeline, root cause, action items, KFM addition.

### 13.7 Pre-deploy checklist

[`templates/pre-deploy-checklist.md`](templates/pre-deploy-checklist.md) — the §7 four-gate checklist.

### 13.8 Quick reference

The [1-page cheatsheet](cheatsheet.md) summarizes the entire handbook on a single page. Print, pin, share.

---

## Companion repositories

This handbook is one of three repos in the AI Quality stack:

- **[awesome-ai-feature-testing](https://github.com/Aftabbs/awesome-ai-feature-testing)** — testing patterns, tools, failure modes by AI feature category. The reference catalog this handbook points to.
- **[claude-code-rules-for-ai-features](https://github.com/Aftabbs/claude-code-rules-for-ai-features)** — opinionated CLAUDE.md / AGENTS.md with 10 commandments for shipping AI features. The rules layer above this handbook's principles.
- **[BehaviorCI](https://github.com/Aftabbs/BehaviourCI)** — the merge-gate tool that implements §6, §7, §8 in practice.

---

## Versioning

This is v1.0 (2026-05-02). Major revisions will be released as v1.1, v1.2, etc., with a changelog. The maturity model and the behavioral-spec format are stable contracts; the templates and the EU AI Act mapping will be updated as the regulations evolve.

## License

Content: CC-BY-SA-4.0. Templates are fillable; you may use them in proprietary projects without restriction. The handbook itself is shareable with attribution.

## Contributing

PRs welcome on:

- New chapters (proposed via issue first)
- Anonymized case studies for the [`case-studies/`](case-studies/) folder
- Translation
- Template improvements

See [CONTRIBUTING.md](CONTRIBUTING.md) for the bar.

---

*If you ship AI features, this handbook exists to make tomorrow's you grateful for today's you. The discipline is the thing. The tools come and go.*

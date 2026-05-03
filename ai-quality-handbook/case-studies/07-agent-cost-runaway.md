# Case study 07 — Sales-research agent spent $4.20 on a single call

> Anonymized. B2B SaaS sales-enablement tool with a research agent.

## What happened

The team shipped a sales-research agent that, given a target company name, produced a 1-page brief with citations. Median cost per call was around $0.18; p95 was around $0.40. The team set a `cost_p95_max_absolute_usd: 0.40` gate.

A user typed "Adidas" as the target. Adidas has thousands of public-news mentions. The agent's web-search tool returned 50 hits per query; the agent ran 3 search queries plus 8 page-fetch calls plus 2 reranker calls plus the synthesis call. The retrieved context for the synthesis call was ~180k tokens. The synthesis call alone cost ~$3.40 at gpt-4o pricing.

The total call cost was $4.21. The user got a brief that was ~1500 words long (oversized) and cited some clearly low-relevance pages.

The cost gate (`p95 ≤ $0.40`) didn't fire because this was a single outlier — the eval bundle's p95 over 50 cases was still $0.39. Production p95 over a 24-hour window was also under $0.40 because the day had 200 normal calls and 1 expensive one.

## Impact

- One $4.21 call (vs. typical $0.18 — ~24x cost spike)
- Three similar incidents per week as users typed in large-corpus targets
- ~$2.3k/week extra spend before the team caught the pattern
- A few user-facing 90-second wait times that exceeded latency expectations

## What was missing

Per HANDBOOK §6.1 and §10:

- **The cost gate was on aggregate p95, not per-call absolute.** A single $4 call doesn't move p95 if the volume is high.
- **No per-call hard cap.** The agent had a tool-call budget (12 calls), but no token-budget at the synthesis step.
- **No retrieved-context cap.** The synthesizer received whatever the agent had gathered.

## What would have caught it

1. **Per-call absolute cost cap** (`cost_per_call_max_absolute_usd: 0.50`). Any single call exceeding $0.50 fails the eval / pages production. (HANDBOOK §6.1 lists this as a P95-floor; this incident motivates upgrading it to a per-call hard cap for cost-volatile features.)
2. **Retrieved-context token cap.** Synthesis step receives at most N tokens; remainder is dropped or rolled into a fallback summarization step.
3. **Pre-synthesis cost prediction.** Before the synthesis call, estimate cost based on token count; if estimated cost > cap, abort the agent run with a "target too broad" error.
4. **Eval cases with broad-corpus targets** — Adidas, Microsoft, Apple, etc. The team's eval was 50 mid-tail companies; broad-corpus cases weren't represented.

## Maturity-model classification

The team was at **L3** for accuracy / latency but had per-call cost-management gaps. After this incident, they moved cost-management to a peer-of-accuracy concern in their CI: per-call cap, per-step token cap, eval cases with broad-corpus inputs.

The Rule 10 entry:

```markdown
N. **Broad-corpus target cost runaway.** Generic / large-corpus target names
   cause the agent to fan out into many search queries with very long retrieved
   contexts; single-call cost can spike 20x.
   - Repro: evals/datasets/known-failures/N-broad-corpus-target.jsonl
   - Guarded by: per-call absolute cost cap, retrieved-context token cap
   - Discovered: 2026-NN-NN
   - Last regression: never
```

## Lesson

Aggregate cost gates miss per-call outliers. Agents have unbounded surface area; without absolute per-call caps, a worst-case input becomes the worst-case bill. For any feature with high cost variance (agents, RAG over heterogeneous corpora, computer-use agents), gate on both aggregate and per-call absolute. Make sure the eval set has cases that *could* trigger a worst-case path.

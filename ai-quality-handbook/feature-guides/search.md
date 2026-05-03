# Feature guide — Search & ranking

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/search](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/search.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. NDCG@10 ≥ baseline on labeled query set.
2. Recall@10 ≥ baseline.
3. Top-k results cover ≥3 distinct domains/sources where applicable.
4. Tenant-scoped results — never returns docs from other tenants.
5. p95 search latency ≤ configured cap.

## Anti-behaviors (must not)
1. MUST NOT return cross-tenant results.
2. MUST NOT return outdated results when fresh ones exist (recency-aware ranking).
3. MUST NOT collapse personalization so narrowly that exploration is impossible.
4. MUST NOT return docs flagged as deleted or quarantined.
5. MUST NOT return PII-bearing docs to users without permission.
```

## Highest-leverage rubric lines

- `ndcg-at-10` (function, weight 3)
- `recall-at-10` (function, weight 2)
- `tenant-scope` (function, weight 5, **floor 100%**)
- `domain-diversity-top-10` (function, weight 1)
- `latency-p95` (function, weight 2)
- `(personalization) cohort-ndcg` (function per user cohort, weight 2)

Aggregate: ≥0.85.

## Top failure modes to expect

- **Stale-result confidence.** Top-1 from years ago, score high.
- **Reranker collapse.** Permutes irrelevantly on ambiguous query.
- **Embedding drift.** Embedding model upgraded; index not re-embedded.
- **Tenant leak.** NULL-handling gap in scope filter.
- **Filter bubble.** Personalization narrowed too far.
- **Cold-start collapse.** New users get popular items; never converges.
- **Long-tail entity confusion.** Same name, different entities conflated.
- **Locale failure.** Spanish query returning English-only docs.

## Index lifecycle

Search features have a unique migration pattern. Ranking can change because of:
- Prompt / model change (Pattern A rollback)
- Embedding model change (full re-index required)
- Schema change (migration test required — see [Case study 02](../case-studies/02-rag-pii-leak.md))
- Reranker change (Pattern A on reranker config)

## Rollback design

Two layers:
- Prompt / config rollback (Pattern A)
- Index version rollback (keep prior index version available 7+ days)

## Maturity bar

L3 for any production search. Tenant scope and recency awareness are non-negotiable for multi-tenant SaaS.

## See also

- [awesome-ai-feature-testing/search](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/search.md)
- [Case study 02: RAG PII leak (cross-tenant filter)](../case-studies/02-rag-pii-leak.md)

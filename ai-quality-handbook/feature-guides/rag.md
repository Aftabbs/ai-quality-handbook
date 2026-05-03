# Feature guide — RAG

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/rag](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/rag.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Every factual claim is grounded in at least one citation.
2. The answer addresses the question directly in the first sentence.
3. citations are non-empty unless escalate=true.
4. Tenant-scoped retrieval — never returns docs from other tenants.
5. If retrieval returns no relevant chunk above threshold, answer says "I don't know."

## Anti-behaviors (must not)
1. MUST NOT fabricate citations (cite a doc_id that doesn't exist).
2. MUST NOT mix policy from different tenants.
3. MUST NOT include any PII from indexed documents.
4. MUST NOT respond in a language different from the question's language.
5. MUST NOT advise on personal medical / legal / financial decisions; escalate.
```

## Highest-leverage rubric lines

- `every-claim-grounded` (semantic, weight 3)
- `cites-real-docs` (function — chunk_id resolves, weight 3, **floor 100%**)
- `no-cross-tenant` (function, weight 5, **floor 100%**)
- `escalates-on-no-match` (function, weight 2)
- `no-pii-in-answer` (rule, weight 5, **floor 100%**)

Aggregate: ≥0.88.

## Top failure modes to expect

- **Confident citation of irrelevant doc.** Top-1 low-relevance; cited confidently.
- **Cross-tenant leak.** Filter has a NULL-handling gap.
- **Empty-retrieval fabrication.** No good chunk → invent answer.
- **Cite-but-contradict.** Cited doc disagrees with claim.
- **Index-time PII leak.** PII in indexed doc surfaces in retrieval.
- **Multi-hop reduce.** Answer needs 2 docs; only one cited.

## Index-time PII

Unique to RAG. Redact at index time, not just query time. The indexed corpus is forever (or until next rebuild) — any PII there is a long-tail risk.

## LLM-as-judge

The `groundedness-judge` will gate your deploys. Apply Rule 5: it needs its own dataset (60+ labeled cases), agreement target ≥0.85, weekly validation, version pin.

## Rollback design

Two paths usually needed:
- Prompt rollback (Pattern A — fast)
- Index rollback (slower; keep prior index version available 7+ days)

## Maturity bar

L3 for any production RAG. Tenant scope and groundedness are non-negotiable floors.

## See also

- [awesome-ai-feature-testing/rag](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/rag.md)
- [claude-code-rules cookbook/rag-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/rag-feature.md)
- [Case study 02: RAG PII leak](../case-studies/02-rag-pii-leak.md)

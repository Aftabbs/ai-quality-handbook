# Case study 02 — Internal RAG returned cross-tenant PII

> Anonymized. Mid-size B2B SaaS company with multi-tenant data.

## What happened

The team shipped an internal-knowledge RAG used by support agents. The vector index was rebuilt nightly from a corpus of customer communications, internal docs, and runbooks. Each doc had a `tenant_id` metadata field; retrieval was supposed to filter by it.

A quarterly index migration moved to a new vector store. The migration script preserved the `tenant_id` field for ~99.7% of docs but silently dropped it for the 0.3% — a backfill of older "shared" docs that the legacy schema didn't include.

The RAG retrieval filter was `WHERE tenant_id = $current OR tenant_id IS NULL` (intent: include "global" docs). After migration, the 0.3% that lost their tenant_id appeared as `NULL` and matched the global filter — making them retrievable across tenants.

A support agent at Tenant A asked a question that retrieved a doc originally written for Tenant B. The doc contained Tenant B's customer email address as an example. The agent didn't realize it was cross-tenant; included the example in their reply to a Tenant A customer.

## Impact

- One concrete cross-tenant PII leak event
- Discovered by Tenant A's customer ("how do you know about a Tenant B customer?")
- Mandatory disclosure to both tenants per their MSA terms
- Two-week internal audit; ten-day index rebuild and remediation

## What was missing

Per HANDBOOK §5 and §6:

- **No PII threat model** for the RAG. Index-time PII redaction was assumed but never audited.
- **No `tenant-scope` rubric line** with a 100% floor.
- **No alerting on schema drift** in the vector store metadata (would have caught the missing `tenant_id` field).
- **No nightly sample re-scan** of the index for cross-tenant matches.

## What would have caught it

1. PII threat model documenting that the RAG retrieval filter is the *primary* tenant-scope guard. Documenting it forces the team to consider what happens when the filter has a gap.
2. Rubric line `no-cross-tenant` as a function check: for every retrieved doc, verify its `tenant_id` matches the query's `tenant_id` or is the explicit `_global` sentinel. NULL is **not** treated as global.
3. CI gate on the migration: a test that runs 50 cross-tenant test queries against the migrated index and asserts no doc is retrieved without a documented `tenant_id`.
4. Production sample: random 100 retrievals/day, audited for tenant boundaries.

## Maturity-model classification

The team was at **L2 — offline rubric** for *content quality* but **L0** on tenant-isolation. After this incident, they moved to L3 (with a tenant-scope behavior floored at 100%) and shipped the migration test.

The Rule 10 entry:

```markdown
N. **Cross-tenant retrieval via missing tenant_id.** Vector store migration dropped
   tenant_id field on 0.3% of docs; retrieval filter treated NULL as 'global'.
   - Repro: evals/datasets/known-failures/N-cross-tenant-null.jsonl
   - Guarded by: rubric behavior `tenant-scope` (floor 100%, weight 5)
   - Discovered: 2026-NN-NN in INC-XXXX
   - Last regression: never
```

## Lesson

For multi-tenant RAG, the tenant filter is not a "best effort" check. NULL must explicitly mean *fail closed* (no match), not "global." Migrations to new vector stores must include a cross-tenant test, not just an "all docs migrated" count.

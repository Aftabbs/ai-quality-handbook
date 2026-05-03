# Changelog

All notable changes to the AI Quality Handbook.

The handbook follows semantic versioning of *content*: minor versions add chapters, sections, templates, or case studies; major versions change the L0–L4 maturity model or the 13-chapter structure.

---

## [1.0.0] — 2026-05-02

Initial public release.

### HANDBOOK.md

- 13 chapters covering shipping mindset, behavioral specs, eval pyramid, the five quality dimensions, PII / prompt injection / OWASP LLM Top 10, pre-merge gates, pre-deploy gates, shadow + canary + rollout, continuous monitoring, incident response, the L0–L4 maturity model, EU AI Act mapping, and templates index.
- The L0–L4 AI Quality Maturity Model.

### Templates

7 fillable templates added:

- behavioral-spec.md
- eval-plan.md
- model-card.md
- pii-threat-model.md
- deployment-ticket.md
- incident-postmortem.md
- pre-deploy-checklist.md

### Case studies

8 anonymized case studies added:

- 01 — Summarizer hallucination on revenue numbers
- 02 — Internal RAG returned cross-tenant PII
- 03 — Prompt change tripled cost; nobody noticed for three days
- 04 — LLM judge drifted; team shipped a 7% accuracy regression
- 05 — Rollback took 22 minutes; 1,800 customers got a wrong reply
- 06 — Aggregate score lied; Spanish segment had regressed 11 points
- 07 — Sales-research agent spent $4.20 on a single call
- 08 — Classifier "improved" 2pp but stopped escalating low-confidence cases

### Feature guides

10 one-pagers added — chatbot, rag, summarizer, code-gen, image-gen, voice, agent, multimodal, classifier, translation, search.

### Cheatsheet

Single-page summary at `cheatsheet.md`.

### EU AI Act mapping

Article-by-article mapping (Articles 9, 10, 11, 12, 13, 14, 15) to handbook artifacts. NIST AI RMF cross-walk included.

---

## [Unreleased]

(Track upcoming changes here.)

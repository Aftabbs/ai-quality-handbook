# The AI Quality Handbook

> A single-file, opinionated guide to shipping AI features safely.

[![License: CC BY-SA 4.0](https://licensebuttons.net/l/by-sa/4.0/80x15.png)](https://creativecommons.org/licenses/by-sa/4.0/)
[![Version: 1.0](https://img.shields.io/badge/Version-1.0-blue)](HANDBOOK.md)
[![EU AI Act-aware](https://img.shields.io/badge/EU%20AI%20Act-aware-ffcc00)](eu-ai-act-mapping.md)  

This is the definitive, single-file guide for the engineering manager, tech lead, or staff engineer who has been asked to ship an AI feature to enterprise — and who knows enough to know they don't yet know how to do it safely.
 
It's also for the AI engineer at a 50-person startup staring at the OWASP LLM Top 10, the EU AI Act, and the third "the AI promised a refund" Slack message this quarter.

---

## What you'll find here

- **[HANDBOOK.md](HANDBOOK.md)** — the canonical artifact. ~8000 words. Thirteen chapters covering shipping mindset, behavioral specs, eval pyramid, the five quality dimensions, PII / OWASP, pre-merge gates, pre-deploy gates, shadow + canary + rollout, continuous monitoring, incident response, the L0–L4 maturity model, EU AI Act mapping, and templates.
- **[cheatsheet.md](cheatsheet.md)** — one page. Pin it. Print it. Share it.
- **[templates/](templates/)** — 7 fillable templates: behavioral spec, eval plan, model card, PII threat model, deployment ticket, incident postmortem, pre-deploy checklist.
- **[case-studies/](case-studies/)** — 8 anonymized real-world incidents with what would have caught them.
- **[feature-guides/](feature-guides/)** — 10 one-pagers for chatbots, RAG, summarizers, code-gen, image-gen, voice, agents, multimodal, classifiers, translation, search.
- **[eu-ai-act-mapping.md](eu-ai-act-mapping.md)** — Article-by-article mapping of EU AI Act requirements to handbook artifacts.

---

## Who this is for

- The tech lead asked to ship an enterprise chatbot
- The engineering manager whose team's AI feature just had its third regression in a quarter
- The AI engineer who knows the tools but is missing the engineering layer above
- The compliance / risk lead who needs a vocabulary for talking to engineering about AI quality
- The team that just adopted [`claude-code-rules-for-ai-features`](https://github.com/Aftabbs/claude-code-rules-for-ai-features) and wants the why behind the rules

## Who it's NOT for

- Researchers benchmarking model quality (use HELM, Inspect AI, Eureka)
- Teams looking for a tool tutorial (use the tool's own docs)
- Anyone wanting a generic "AI ethics" overview (this handbook is engineering-first)

---

## How to use it

### As a learner

Read [HANDBOOK.md](HANDBOOK.md) end to end (≈45 minutes). Then return to specific chapters as you face the problems.

### As a tech lead

Pin [cheatsheet.md](cheatsheet.md) to the team's wiki. Use it for onboarding new hires. Use the templates for any new AI feature.

### As a compliance / risk lead

Read [eu-ai-act-mapping.md](eu-ai-act-mapping.md). Map your existing features to the articles. Use the model-card and PII threat model templates as your documentation primitives.

### As a team that already has running AI features

Run the maturity self-assessment (HANDBOOK §11.6). Pick the lowest-scoring feature; pull it up one level. Repeat quarterly.

---

## Part of the AI Quality stack

This handbook is one of three repos in the AI Quality stack:

- **[awesome-ai-feature-testing](https://github.com/Aftabbs/awesome-ai-feature-testing)** — testing patterns, tools, failure modes by AI feature category. The reference catalog this handbook points to.
- **[claude-code-rules-for-ai-features](https://github.com/Aftabbs/claude-code-rules-for-ai-features)** — opinionated CLAUDE.md / AGENTS.md with 10 commandments for shipping AI features. The rules layer above this handbook's principles.
- **[BehaviorCI](https://github.com/Aftabbs/BehaviourCI)** — the merge-gate tool that implements §6, §7, §8 in practice.

---

## Versioning

v1.0 (2026-05-02). The maturity model and behavioral-spec format are stable contracts; the templates and EU AI Act mapping will evolve as the regulations themselves do.

[Changelog](CHANGELOG.md).

---

## Contributing

PRs welcome on:

- New chapters (propose via issue first)
- Anonymized case studies for the [`case-studies/`](case-studies/) folder
- Translations
- Template improvements

We are conservative about:

- Changes to the L0–L4 maturity model (this is a stable contract)
- Changes to the 13 chapter structure (we discuss for ≥14 days)
- New templates (must serve a section of the handbook, not invent new requirements)

See [CONTRIBUTING.md](CONTRIBUTING.md).

---

## License

Content: **CC-BY-SA-4.0**.

You may copy, modify, share with credit + share-alike. The templates may be used in proprietary projects without restriction. The handbook itself is shareable with attribution.

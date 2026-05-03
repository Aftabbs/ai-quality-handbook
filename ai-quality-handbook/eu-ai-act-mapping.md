# EU AI Act mapping — what to actually do

> Per HANDBOOK §12. Maps EU AI Act articles to concrete artifacts in this handbook + companion repos.

The EU AI Act enters full application in stages: prohibitions and AI literacy from Feb 2025, GPAI obligations from Aug 2025, high-risk system rules from **Aug 2026**.

If your feature is high-risk (employment, education, credit, law enforcement, infrastructure, etc.), you owe documented compliance. The good news: the engineering practices in this handbook map cleanly to the technical articles. The bad news: you must document them.

This page is the translation layer.

---

## Article-by-article

### Art. 9 — Risk management system

**The Act says:** Establish a continuous risk management process across the lifecycle of the high-risk AI system. Identify and analyze foreseeable risks. Adopt risk management measures.

**You owe:**
- A documented risk register listing foreseeable failure modes and mitigations.

**This handbook maps to:**
- The behavioral spec's `## Anti-behaviors` section (foreseeable harm)
- The `## Known failure modes` section (discovered harm with mitigation)
- The eval rubric's `per-behavior-floor` lines (high-cost violations gated at 100%)

**Concrete artifact:** Your spec + rubric + failure-mode catalog *is* your Art. 9 risk management documentation. Format the spec, hand it to a regulator, point at the failure modes section.

---

### Art. 10 — Data and data governance

**The Act says:** Training, validation, and testing datasets are subject to data governance practices. Datasets must be relevant, representative, free of errors, and complete.

**You owe:**
- Data provenance documentation
- Bias and stratification disclosure
- Error rates and known limitations

**This handbook maps to:**
- The eval-plan template (§13.2) — dataset stratification, sources, adversarial coverage
- The eval dataset's frontmatter (provenance, count, last_changed)

**Concrete artifact:** `evals/datasets/<feature>.jsonl` with documented provenance + the eval plan describing how it was assembled.

---

### Art. 11 — Technical documentation

**The Act says:** Technical documentation drawn up before placing on the market, kept up-to-date.

**You owe:**
- A model card per feature
- Documentation of architecture, intended use, performance characteristics, known limitations

**This handbook maps to:**
- The model card template (§13.3)
- The behavioral spec
- The eval baseline JSON (formal record of measured performance)

**Concrete artifact:** `docs/model-cards/<feature>.md`. Updated on every model swap.

---

### Art. 12 — Record-keeping (logging)

**The Act says:** High-risk AI systems must automatically log events relevant to identifying risks, monitoring operation, and post-market monitoring.

**You owe:**
- Per-call logs of input, output, model version, prompt SHA
- Retention policy (typically 6 months for high-risk)
- Auditability

**This handbook maps to:**
- The pre-deploy gate that locks `evals/baselines/<feature>.json` per merge
- The deploy log entry on every release (timestamp, prompt SHA, model id, baseline SHA, approver)
- Production observability (Phoenix / Langfuse / Opik tracing)

**Concrete artifact:** Your CI deploy log + your production tracing system. PII-redact the inputs/outputs per §5.1; retain trace metadata 180+ days.

---

### Art. 13 — Transparency and information for users

**The Act says:** High-risk AI systems must be designed and developed in such a way to ensure operation is sufficiently transparent.

**You owe:**
- Clear user-facing disclosure that AI is involved
- Information about capabilities and limitations
- Instructions for use

**This handbook maps to:**
- UI disclosure copy (cite the relevant article)
- The model card's "Intended use" + "Known limitations" sections

**Concrete artifact:** A short, visible label: "Suggested by AI; verify before sending." plus a help link to the model card or limitations page.

---

### Art. 14 — Human oversight

**The Act says:** High-risk AI systems must be designed to allow effective oversight by natural persons during the period of use.

**You owe:**
- A meaningful human-in-the-loop for consequential decisions
- The ability for the human to override or stop the AI
- Documented escalation triggers

**This handbook maps to:**
- The "escalate-on-low-confidence" pattern in the spec
- The destructive-intent confirmation rule
- The rollback path (§10) — "stop the AI" at the system level

**Concrete artifact:** The escalation policy in your spec + the rollback drill log. Both are auditable.

---

### Art. 15 — Accuracy, robustness, and cybersecurity

**The Act says:** High-risk AI systems must achieve an appropriate level of accuracy, robustness, and cybersecurity, and perform consistently throughout their lifecycle.

**You owe:**
- Documented performance metrics
- Robustness testing (adversarial inputs, concept drift)
- Cybersecurity controls (against poisoning, evasion, model inversion, etc.)

**This handbook maps to:**
- The eval bundle (§3) — measured performance per the rubric
- Adversarial dataset coverage (§3.5) — robustness via 1/3 adversarial cases
- The OWASP LLM Top 10 mitigations (§5.3) — cybersecurity layer
- Continuous monitoring + drift detection (§9) — lifecycle consistency

**Concrete artifact:** Your eval baselines, your adversarial sub-dataset, your PII threat model, your prompt-injection mitigation list. All are auditable.

---

## What "high-risk" means

Annex III of the Act lists the high-risk use cases. Common ones:

- **Employment** — recruitment, evaluation, promotion, termination
- **Education and vocational training** — admission, evaluation, monitoring
- **Access to essential services** — credit scoring, public benefits, insurance pricing
- **Law enforcement** — risk assessment, evidence assessment
- **Migration, asylum, border control**
- **Administration of justice and democratic processes**
- **Critical infrastructure** — safety in transportation, energy, water

If your AI feature is in one of these categories, you're high-risk. If it's adjacent (e.g. "we screen support tickets that might trigger an insurance claim"), get legal advice.

If it's neither — it's still good practice to follow this handbook, and the documentation falls out for free if a regulator ever asks.

---

## NIST AI RMF mapping (US complement)

| NIST function | EU AI Act analog | Handbook section |
|---|---|---|
| Govern | (no direct analog; cultural) | §11 (maturity model) + RACI |
| Map | Art. 9 | §2 (spec), §5 (PII threat model) |
| Measure | Art. 10, Art. 15 | §3 (eval pyramid), §4 (5 quality dimensions), §9 (continuous) |
| Manage | Art. 14 | §10 (incident response, rollback) |

---

## Other regimes

- **ISO/IEC 42001** — AI management system standard. Roughly aligned with NIST RMF + EU AI Act. Increasingly required for procurement.
- **Colorado AI Act** (US, 2026) — for "high-risk AI" in employment / consumer-facing.
- **NYC Local Law 144** — automated employment decision tools.
- **Illinois BIPA-style** — biometric data handling.
- **GDPR** — for any feature touching EU residents' personal data; this handbook's §5.1 (PII redaction) is your primary GDPR posture for AI.

The pattern is consistent across regimes: documented spec, measurable behavior, audit trail, human oversight, incident response. Different vocabularies; same engineering substance.

---

## What to do this quarter

If you ship anything that could be classified as high-risk and the EU AI Act high-risk articles activate Aug 2026:

1. **Stand up a model registry log.** Every deploy: timestamp, prompt SHA, model id, baseline SHA, approver. (1 day of work.)
2. **Ship a model card per feature.** Use the template (§13.3). (Per feature: 30 minutes once you have a spec.)
3. **Wire UI disclosures.** A short, visible label on AI-influenced output. (Per surface: 1 day.)
4. **Document escalation triggers.** When does this AI feature route to a human? Put it in the spec. (Per feature: 30 minutes.)

These four together cover most of Art. 11–14. Art. 9 / 10 / 15 fall out of doing the §1–§9 engineering work properly.

The cost is mostly paperwork. The substance is the engineering you already need to do to ship AI safely.

# Feature guide — Chatbots

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/chatbots.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Address the user's stated issue in the first sentence.
2. Match the language detected on input.
3. Stay under <length cap>.
4. Use the configured persona; do not drift toward user's tone.
5. Escalate to a human when confidence < threshold OR sentiment is hostile.

## Anti-behaviors (must not)
1. MUST NOT promise refunds, credits, compensation (or domain-specific equiv).
2. MUST NOT include any PII from prior tickets / sessions.
3. MUST NOT respond in a different language from the inbound.
4. MUST NOT auto-confirm destructive actions without user verification.
5. MUST NOT include exclamation marks / marketing language (or your equiv).
```

## Highest-leverage rubric lines

- `addresses-question-directly` (semantic, weight 2)
- `matches-language` (rule, weight 1)
- `no-pii-in-output` (rule, weight 5, **floor 100%**)
- `no-promised-refunds` or your domain anti-action (must-not-contain, weight 5, **floor 100%**)
- `professional-tone` (semantic, weight 2)

Aggregate: ≥0.85.

## Top failure modes to expect

- **Sycophancy spiral.** Apology language escalates each turn.
- **Refusal cascade.** One borderline refusal poisons subsequent turns.
- **Code-switch lock.** Multilingual bot locks onto one language.
- **Persona drift.** Multi-turn conversation drifts toward user's tone.
- **Auto-confirm dangerous action.** "Cancel my account" → "Done."

## Rollback design

Pattern A (feature flag) is standard. Cost: low. Drill quarterly.

## Maturity bar

L2 minimum (CI-gated rubric). L3 if customer-facing in regulated domain (finance, healthcare).

## See also

- [awesome-ai-feature-testing/chatbots](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/chatbots.md) — full pattern catalog and tool list
- [claude-code-rules cookbook/chatbot-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/chatbot-feature.md) — full worked example

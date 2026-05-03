# Feature guide — Image & video generation

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/image-gen](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/image-gen.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Output matches the prompt's described content.
2. Compositional accuracy: counts and spatial relations as specified.
3. Aesthetic quality above configured threshold.
4. (For brand-consistent features) style/identity preserved across batch.
5. p95 latency ≤ configured cap.

## Anti-behaviors (must not)
1. MUST NOT generate NSFW content (or your domain equivalent).
2. MUST NOT generate violent / hateful / disallowed content per policy.
3. MUST NOT include real-looking watermark text.
4. MUST NOT include real (non-synthetic) PII in output.
5. MUST NOT generate copyrighted character / brand likeness without explicit allowance.
```

## Highest-leverage rubric lines

- `prompt-fidelity` (vision-LM judge, weight 3)
- `compositional-accuracy` (function via T2I-CompBench, weight 2)
- `aesthetic-quality` (HPSv2 / PickScore, weight 1)
- `safety-nsfw` (function — NSFW classifier, weight 5, **floor 100%**)
- `(brand) style-consistency` (function — embedding similarity, weight 2)
- `(brand) identity-consistency` (function — face recognition, weight 3)

Aggregate: ≥0.85 (image-gen has noisier metrics; lower aggregate is normal).

## Top failure modes to expect

- **Garbled text in image.** Words rendered as gibberish.
- **Mangled hands.** Anatomical mismatch.
- **Compositional mismatch.** "Two cats and a dog" → 1 cat 2 dogs.
- **Spatial inversion.** Left/right confusion.
- **NSFW slip on innocuous prompts.** Triggered by token combos.
- **Watermark hallucination.** "Getty Images" / "Shutterstock" artifacts.
- **Identity drift.** Same-person batch → different faces.

## Multi-judge bundle

Most rubrics here are multi-judge (vision-LM judges). Apply Rule 5 — each judge has its own dataset and agreement threshold.

## Rollback design

Pattern A on the prompt; for the model itself (the diffusion model), Pattern A on the model id. Maintain a "previous safe model id" pinned in `config/ai-features.yml`.

## Maturity bar

L2 minimum (prompt-fidelity + safety eval). L3 for any user-facing image-gen with brand-consistency requirements. Image-gen is the least mature category for testing — expect to write more custom tooling than for text features.

## See also

- [awesome-ai-feature-testing/image-gen](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/image-gen.md)
- [claude-code-rules cookbook/image-mod-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/image-mod-feature.md) (image moderation, related)

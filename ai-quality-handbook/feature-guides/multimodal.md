# Feature guide — Multimodal & vision

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/multimodal](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/multimodal.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. VQA / caption answer references content actually in the image.
2. Counting questions: count is correct (within tolerance).
3. Spatial questions: relations correct (left/right, above/below).
4. (For document understanding) extracted fields validate against schema.
5. p95 latency ≤ configured cap.

## Anti-behaviors (must not)
1. MUST NOT claim to see things not in the image.
2. MUST NOT invent text it "sees" in image (when image has no text).
3. MUST NOT respond confidently when image is blurry / out-of-distribution.
4. MUST NOT include any PII detected in the image (faces, IDs, license plates).
5. MUST NOT classify medical / legal / financial implications without escalation.
```

## Highest-leverage rubric lines

- `vqa-accuracy` (function vs labeled set, weight 3)
- `object-hallucination` (POPE-style, weight 3, **floor 95%**)
- `spatial-reasoning` (function, weight 2)
- `text-in-image-correct` (function — OCR + match, weight 2)
- `(documents) field-accuracy` (function, weight 2)
- `pii-not-leaked` (rule, weight 5, **floor 100%**)
- `bias-fairness` (function across demographic segments, weight 2)

Aggregate: ≥0.87.

## Top failure modes to expect

- **Object hallucination.** Claims things not in image.
- **Counting failure.** Off-by-N.
- **Text-in-image hallucination.** Invents text on text-less image.
- **Spatial inversion.** Left/right confusion.
- **Color drift.** Confidently wrong color.
- **Aspect-ratio sensitivity.** Same image cropped differently → different answer.
- **Diagram-vs-photograph confusion.** Misclassifies UI screenshots as photos.

## Multi-judge bundle

Multimodal evals lean on vision-LM judges. Apply Rule 5: judge has its own eval set.

## Document understanding

Treat this as a hybrid of multimodal + classifier (structured extractor). Per-field accuracy + schema validity.

## Rollback design

Pattern A on prompt + model id.

## Maturity bar

L2 minimum. L3 for production VQA / document AI. Multimodal testing is mid-maturity; expect more custom tooling than text features.

## See also

- [awesome-ai-feature-testing/multimodal](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/multimodal.md)

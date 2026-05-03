# Contributing

The AI Quality Handbook is an opinionated single-file artifact. Edits are welcome but the bar is high — a misleading sentence here can land in a real production playbook somewhere.

## What we accept

### Anonymized case studies

If you've experienced an AI feature incident, write it up using the format of the existing entries in [`case-studies/`](case-studies/). Anonymize: no company names, no customer text, no specific dollar amounts that identify a company.

### Template improvements

If a template fails on a real edge case, propose a fix. Show the case (anonymized) and propose the change.

### EU AI Act / NIST AI RMF mapping updates

Regulations change. If a section is wrong or outdated as of the merge date, propose a fix with a citation.

### Translations

PRs welcome to add `HANDBOOK.<lang>.md`, `cheatsheet.<lang>.md`, etc.

## What we DON'T accept (without discussion)

### New chapters

The 13-chapter structure is a stable contract. Add a chapter only after a 14-day discussion in an issue. New chapters require either reorganizing existing content or making the case for a genuinely new topic.

### Changes to the L0–L4 maturity model

This is the most-cited part of the handbook. Changing the model breaks every existing self-assessment. The bar is very high.

### "I think the handbook should also cover X"

If X is a tool, see [awesome-ai-feature-testing](https://github.com/Aftabbs/awesome-ai-feature-testing) — that's the tool catalog.
If X is a methodology rule, see [claude-code-rules-for-ai-features](https://github.com/Aftabbs/claude-code-rules-for-ai-features) — that's the rules layer.
The handbook covers principles, the maturity model, processes, and templates. Other content goes in the companion repos.

### Marketing language

"State-of-the-art", "industry-leading", "world's first" — get rejected on PR review. Plain prose only.

## Style

- Imperative voice for principles.
- Numbered lists where order matters; bullet lists otherwise.
- ≤3 nested levels of headers in templates.
- Code blocks ≤30 lines; fence and label.
- No emojis.
- Per-chapter word budget: ~600 words. Long chapters are a smell.

## PR checklist

- [ ] No emojis added
- [ ] No marketing language
- [ ] Edits to HANDBOOK.md are reflected in the cheatsheet (if relevant)
- [ ] New case studies are anonymized
- [ ] Links to companion repos use the canonical paths
- [ ] EU AI Act mapping cites the article number when relevant

## Code of conduct

Disagreement is welcome via PR or issue. The handbook is opinionated; we will reject PRs whose change weakens an opinion without strong reasoning. If your case is "I disagree", write up the case in an issue rather than a PR.

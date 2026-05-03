# <Feature name> — behavioral spec

> Fill this template per feature. Lives at `specs/<feature>.md`.
> Five bullets per section, max. If you need more, you have two features.

## Purpose

One sentence. Why this feature exists. Who it serves.

(Example: "Drafts one-paragraph reply suggestions for Tier-1 support agents reading inbound emails. Reduces median reply time from 8 minutes to 90 seconds.")

## Inputs

What the feature receives. Shape, types, examples. Note any PII fields explicitly.

- field_1 (type, constraint, PII?)
- field_2 (type, constraint, PII?)
- ...

## Outputs

What the feature produces. Shape, types, format constraints.

- field_1 (type, constraint)
- field_2 (type, constraint)
- ...

## Behaviors (must)

≤5 numbered, testable behaviors. Imperative, declarative, concrete.

1. ...
2. ...
3. ...

## Anti-behaviors (must not)

≤5 numbered, testable anti-behaviors. Absolute.

1. MUST NOT ...
2. MUST NOT ...
3. ...

## Known failure modes

Numbered. Empty on day 1. Grows with every incident.

(Format per Rule 10 of the rules repo:)

```markdown
1. **<Name>.** <One-line description>
   - Repro: <path-to-jsonl>
   - Guarded by: <rubric-behavior-id>
   - Discovered: <YYYY-MM-DD> in <incident-id>
   - Last regression: <date or "never">
```

---

## Approval

- Author: @
- Reviewed by: @
- Approved on: YYYY-MM-DD

# Feature guide — Code-gen

> One-pager. For depth: [HANDBOOK.md](../HANDBOOK.md). For tools and patterns: [awesome-ai-feature-testing/code-gen](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/code-gen.md).

## Spec template (5 must / 5 must-not)

```markdown
## Behaviors (must)
1. Generated code compiles and lints clean.
2. Generated tests run and pass against a fresh environment.
3. All cross-references inside generated files are consistent.
4. README documents how to run, test, deploy.
5. Dependencies are pinned to specific versions.

## Anti-behaviors (must not)
1. MUST NOT include hardcoded credentials, API keys, or production URLs.
2. MUST NOT include `eval()` / `exec()` / `shell=True` unless explicitly requested.
3. MUST NOT generate code with known-vulnerable dependency versions.
4. MUST NOT generate code that bypasses authn/authz on public endpoints.
5. MUST NOT skip writing the test file — every endpoint has at least one test.
```

## Highest-leverage rubric lines

Code-gen leans into deterministic checks (function type), unlike most AI features.

- `compiles-clean` (function, weight 3, **floor 100%**)
- `lints-clean` (function, weight 2)
- `tests-pass` (function, weight 3, **floor 100%**)
- `cross-refs-resolve` (function, weight 3)
- `no-secrets` (regex, weight 5, **floor 100%**)
- `no-unsafe-primitives` (regex, weight 5, **floor 100%**)
- `vuln-deps` (function — OSV lookup, weight 3)
- `includes-test-file` (function, weight 2)

Aggregate: ≥0.95.

## Top failure modes to expect

- **Hallucinated import.** `from foo.bar import X` where X doesn't exist.
- **Test with `assert True`.** Compiles, runs, passes, tests nothing.
- **Migration that doesn't roll forward.** Syntactic-fine; semantically broken.
- **API-key in example.** Real-looking test key.
- **Cross-framework drift.** Asked Flask, generated FastAPI snippets.
- **Pydantic v1 in v2 codebase.** Wrong major version.
- **Test order dependence.** Pass alone, fail in suite.

## Sandboxing

Hermetic sandbox per case. Real compiler, real linter, real test runner. The eval is the integration test; treat it as such.

## Maturity bar

L3. Code-gen has the highest aggregate threshold of any feature category because shipping bad code = service down.

## See also

- [awesome-ai-feature-testing/code-gen](https://github.com/Aftabbs/awesome-ai-feature-testing/blob/main/features/code-gen.md)
- [claude-code-rules cookbook/code-gen-feature](https://github.com/Aftabbs/claude-code-rules-for-ai-features/blob/main/cookbook/code-gen-feature.md)

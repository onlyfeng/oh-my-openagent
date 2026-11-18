# copy/ - Copy Detection Rules

**Score 9** (21 files, ~1.4k LOC, own barrel + `support/` helpers). One of the six rule categories in [`../AGENTS.md`](../AGENTS.md) (which owns the catalog-level facts: `ALL_RULES` order, rule types, overrides, attribution, and the authoring rules - one rule per file, colocated tests, `support/` for shared helpers). This file maps THIS category's rules.

## RULES (`index.ts` -> `COPY_RULES: readonly PageRule[]`, in registration order)

| Rule | Sev | Detects |
|------|-----|---------|
| `em-dash-overuse.ts` | advisory | ≥8 prose dashes at one-per-500-chars density; has a source-engine twin in `../source/` |
| `marketing-buzzword.ts` | warning | Generic promotion standing in for a concrete product description |
| `aphoristic-cadence.ts` | warning | ≥3 short rebuttals giving unrelated sections the same cadence |
| `theater-slop-phrase.ts` | advisory | Calling a practice "theater" to dismiss it without naming the defect |
| `numbered-section-labels.ts` | advisory | Tiny styled numeric indices on multiple headings - scaffolding, not information |
| `repeated-container-text.ts` | warning | One card repeating a short string in ≥3 different structural slots |
| `skipped-heading.ts` | warning | A heading jumping over a level in the visible outline (largest rule, 205 lines) |
| `broken-image.ts` | warning | An image with no usable source attribute |
| `script-error.ts` | error | An uncaught exception or rejected promise while the page loaded |

All nine are page-engine rules over the rendered `PageSnapshot`; prose extraction comes from `support/page-text.ts`, numeric-index detection from `support/numbered-label.ts` (both non-rule helpers - no `Rule` exports).

## NOTES

- `script-error` is the category's only `error` severity: it signals a broken page, not slop.
- Copy rules key off visible text, so fixture snapshots (`../../testing/build-snapshot`) must include realistic prose runs; `repeated-container-text` and `skipped-heading` tests are the ones to copy when adding a rule.

## COMMANDS

```bash
bun test packages/shared-skills/skills/frontend/scripts/detect/rules/copy
```

Parent: [`../AGENTS.md`](../AGENTS.md).

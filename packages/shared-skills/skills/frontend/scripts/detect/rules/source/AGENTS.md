# source — Source-Rule Family of the Detect Catalog

**Score:** 16 (44 files, 2k LOC, >20 files, own barrel `SOURCE_RULES`). Parent [`rules/AGENTS.md`](../AGENTS.md) covers the six-category aggregate (`ALL_RULES`), the `RuleMeta` contract, and the "no shared helpers next to rules" rule.

## OVERVIEW

The 16 `SourceRule`s that lint raw file text (no rendering): `ai-color-palette`, `aphoristic-cadence`, `border-accent-on-rounded`, `bounce-easing`, `broken-image`, `codex-grid-background`, `dark-glow`, `em-dash-overuse`, `gradient-text`, `gray-on-color`, `marketing-buzzword`, `marquee`, `monotonous-spacing`, `overused-font`, `radial-halo`, `side-tab`. `index.ts` folds them into `SOURCE_RULES: readonly SourceRule[]` in a deliberate order.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add a source rule | One `<rule-name>.ts` + its entry in `index.ts` + colocated `<rule-name>.test.ts` |
| Shared scanning helpers | `support/` — CSS scope/declaration helpers, source text passes, authored color/shadow extraction, image/markup scope helpers, corner scans, accent scans, `checkSource` |
| Aggregate | `index.ts` (`SOURCE_RULES`) — consumed by `../../lint.ts` via `ALL_RULES` |

## CONVENTIONS

- One concern per module; a rule reads like its name. Anything two rules would both do belongs in `support/`.
- `ai-color-palette` and `em-dash-overuse` are source twins of page rules in other categories — keep the two engines' findings semantics aligned when either changes.
- Rule order inside `SOURCE_RULES` is deliberate (lint output order follows it).

## ANTI-PATTERNS

- NEVER add a rule to `index.ts` without its module (unreachable) or without its test (does not land).
- NEVER put a `Rule` export in `support/` — it holds no rules by contract.

## COMMANDS

```bash
bun test packages/shared-skills/skills/frontend/scripts/detect/rules/source
```

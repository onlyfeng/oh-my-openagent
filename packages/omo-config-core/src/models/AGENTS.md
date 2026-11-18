# src/models - Model Catalog, Exclusion, Distribution

**Generated:** 2026-11-18 | c2691818a9

`omo.json` model-catalog helpers: exclusion patterns, cycle detection in catalog references, model-reference resolution, and the per-category model distributor. Distinct domain inside `omo-config-core`: the only place model selection arithmetic lives.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `excluded-models.ts` | `excludedModelPattern`, `isExcludedModel` - glob-style exclusion of a model id. |
| `model-catalog-cycles.ts` | `findModelCatalogCycles` - detect cycles in the user-declared catalog graph. |
| `model-reference-resolution.ts` | `resolveModelReferences` - resolve a reference (catalog entry, alias, provider/model) to a concrete model. |
| `models-distribution.ts` | `createModelDistributor` - turn the resolved catalog into per-category chains. |
| `index.ts` | Barrel re-export. |

## CONVENTIONS

- Tests are co-located `*.test.ts` under `bun test`.
- All four helpers are pure (no I/O); they take the parsed `omo.json` and return typed results.
- `excludedModelPattern` is glob-style: `*` crosses `/`, matching is case-insensitive.
- `createModelDistributor` supports `sequential` and `round_robin` strategies.
- `findModelCatalogCycles` returns typed diagnostics; the loader collects them rather than throwing.

## ANTI-PATTERNS

- Do not duplicate the exclusion/cycle/distribution logic in an adapter - this module is the single owner.

## COMMANDS

```bash
bun test packages/omo-config-core/src/models
bun run --cwd packages/omo-config-core typecheck
```

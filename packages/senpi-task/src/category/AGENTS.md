# src/category/ - Delegate Category Model Resolution

**Score:** 14 (31 files, ~4.6k LOC, ~2/3 tests; own barrel + three package subpaths; `resolveCategory` imported across `agents/`, `runners/`, `manager/`, tools).

## OVERVIEW

Builtin category definitions per provider family plus the resolver producing `ResolvedChildSpec` and the discriminated-union `CategoryResolutionResult` (`resolved` | `disabled` | `not_found` | `model_unavailable`). Gate semantics (`requiresModel`, gateway re-publish rule #9146, chain viability, prompt-append hooks) are owned by [`../../AGENTS.md`](../../AGENTS.md) and not repeated here.

## WHERE TO LOOK

| File | Role |
|------|------|
| `builtins.ts` | `BUILTIN_CATEGORY_DEFAULTS`, `BUILTIN_CATEGORY_REQUIRES_MODEL`, `CATEGORY_DESCRIPTIONS` / `CATEGORY_CALLER_GUIDANCE` / `CATEGORY_PROMPT_APPENDS`; gate helpers `categoryGateModels`, `isCategoryGateSatisfied`, `isCategoryChainViable`, `isCategoryChainRungResolvable` |
| `resolver.ts` | `resolveCategory`, `resolveAvailableCategoryNames`, `builtinCategoryChainCandidates` (294 LOC: model selection + fallback + gate + diagnostics) |
| `{anthropic,openai,google,kimi}-categories.ts` | Per-family tables; `openai-categories.ts` carries the per-model `resolvePromptAppend` hooks |
| `fallback-chains.ts` | Category fallback chains, hand-mirrored from model-core and kept local (see anti-patterns) |
| `coverage.ts` | `resolveCategoryCoverage` (subpath `./category-coverage`) |
| `registry-models.ts`, `user-candidates.ts`, `available-names.ts`, `chain-diagnostics.ts` | Registry port helpers, user-config candidates, listing, diagnostics |
| `types.ts` | `SenpiModelPort`, `SenpiModelRegistryPort` - narrow consumer-defined ports over the model registry |

## CONVENTIONS

- Results are discriminated unions with a `kind` field, never exceptions.
- Categories carry LLM prompt content (`promptAppend`, `callerGuidance`, `description`) next to code - the DO NOT/NEVER strings inside the category files are prompt text shipped to models, not developer rules.
- Gates and chain rungs match live-registry models as `provider/id` strings with exact family/version match; the provider is part of the check.
- Package subpaths: `./category-builtins` (builtins), `./category-resolver` (resolver), `./category-coverage` (coverage); the package barrel re-exports the rest.

## ANTI-PATTERNS

- `fallback-chains.test.ts:5`: this test file must NEVER import chain data from `@oh-my-opencode/model-core` - chain data stays local to this directory (coupling guard).
- Do not widen `SenpiModelPort`/`SenpiModelRegistryPort` beyond what resolution needs; they are consumer-defined seams.

## COMMANDS

```bash
bun test packages/senpi-task/src/category/
bun packages/senpi-task/scripts/manual-category-qa.ts <evidence-dir>
```

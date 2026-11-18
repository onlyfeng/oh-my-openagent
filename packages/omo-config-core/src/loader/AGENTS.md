# src/loader/ — Layered omo.json[c] Loader

## OVERVIEW

27 TypeScript files, ~3.4k LOC. Walked config discovery + layered merge +
validation + diagnostics for `omo.json[c]`; produces the resolved view/profile
consumed by every harness adapter. Barrel `index.ts` re-exports loaders, path
resolvers, merge/prune helpers, and diagnostics types. Parent:
[`packages/omo-config-core/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `loader.ts` | `loadOmoConfig` runtime entry; orchestration of layers |
| `paths.ts` / `resolution.ts` | `resolveOmoConfigPaths`, `resolveHomeDir`, user/project path discovery |
| `layer-validation.ts` | Per-layer schema validation |
| `merge.ts` | Layer merge (user first, then project farthest-to-nearest) |
| `prune-invalid-leaves.ts` | Drop invalid leaves instead of failing the whole load |
| `diagnostic-lines.ts` | Human/machine diagnostics lines |
| `disabled-skills.ts` | Disabled-skill collector |
| `loader.test.ts`, `merge.test.ts`, `paths.test.ts` | Colocated suites |

## CONVENTIONS

- All IO is injected through filesystem ports; no direct `node:fs` in loader
  logic.
- JSONC is accepted everywhere; layers merge recursively with unsafe
  prototype keys stripped during merge.
- Defaults are applied after merging and view selection; views/profiles
  (`resolveOmoConfigView`, `resolveOmoProfileName`) never mutate the merged
  base.
- Internal helpers stay out of the public barrel.

## ANTI-PATTERNS

- Never merge without stripping `__proto__` / `prototype` / `constructor`.
- Do not surface hard errors for a single invalid layer when
  `prune-invalid-leaves` semantics apply — collect diagnostics.

## QA

```bash
tsgo --noEmit -p packages/omo-config-core/tsconfig.json
bun test packages/omo-config-core
```

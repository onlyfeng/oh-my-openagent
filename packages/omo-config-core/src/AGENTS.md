# src/ - config-core source map

**Score 16** (122 TS files, 6 subdirs, cross-adapter barrel). Package semantics - schema field defaults, loader precedence and diagnostics, writer guarantees, migration engine, public API, consumers, generated schema - live ONE LEVEL UP in [`../AGENTS.md`](../AGENTS.md). This file is only the src-internal file map and does not repeat them.

## STRUCTURE

| Subdir | Prod/Test | Owns |
|--------|-----------|------|
| `schema/` | 26/25 | One zod file per config domain, star-barreled by `index.ts`. Files beyond the parent's itemization: `computer.ts`, `device-service.ts`, `provenance.ts` (workspace git-owner allowlist), `model-pad.ts` (2x2 model-profile lane pad - presentation-only, the engine never reads it), `model-profile.ts`, `model-ref.ts`, `reasoning-vocabulary.ts` (`REASONING_LEVELS` + reasoning-suffix split), `session-wake.ts` (#13037), `memory.ts` + `memory-agreements.ts`, `telemetry.ts`, `format-on-mutation.ts`, `legacy-category-names.ts` / `legacy-harness-names.ts` (alias preprocessing), `memory.test-support.ts`. |
| `loader/` | 10/17 | Parent itemizes `loader`/`paths`/`resolution`/`merge`/`types`. Also here: `layer-validation.ts` + `prune-invalid-leaves.ts` (smallest-failing-subtree invalid-value pruning), `diagnostic-lines.ts` (doctor-rendered lines), `disabled-skills.ts` (`disabled_skills` collection across raw layers incl. legacy harness aliases). |
| `migration/` | 13/11 | Parent itemizes `batch`/`engine`/`lock`/`recovery`/`merge`/`commit`/`predicate`. Also here: `journal.ts`, `diff-edits.ts` (deep-equal diff into `ConfigEdit`s), `backup-move.ts` (EXDEV-aware rename→copy fallback), `migration-test-support.ts`, `types.ts`. |
| `models/` | 5/3 | Parent itemizes `model-reference-resolution` + `model-catalog-cycles`. Also here: `excluded-models.ts` (`exclude_models` globs: `*` crosses `/`, matching is case-insensitive), `models-distribution.ts` (`sequential` / `round_robin` distributor). |
| `writer/` | 4/3 | Fully itemized in the parent (`writer`, `surgical-edit`, `types`). |
| `internal/` | 4/0 | In-package primitives, NOT imports from utils: `isPlainObject` / `isUnsafeObjectKey`, `parseJsoncSafe` (jsonc-parser with BOM strip + offset-positioned errors), `toPosixPath` / `posixJoin` - the migration engine's injected FS ports are keyed by forward-slash paths, so Windows backslashes normalize here, never through `node:path` joins. |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Add a config field | The domain file under `schema/` + the root shape in `schema/config.ts`; keep `OmoConfigLayerSchema` partial-variant in sync |
| Accept/reject behavior of unknown keys | `loader/layer-validation.ts` (parent §Layer precedence) |
| Legacy alias → canonical name | `schema/legacy-category-names.ts` / `schema/legacy-harness-names.ts` |
| Path normalization for migration targets | `internal/posix-path.ts` |

## CONVENTIONS

- Tests colocated per subdir (`*.test.ts`); `internal/` is the only subdir without its own tests (exercised through its consumers).
- Every subdir has an `index.ts` barrel; `src/index.ts` re-exports exactly `schema`, `loader`, `models`, `writer`, `migration` - `internal/` is deliberately NOT part of the public surface.
- Schema files stay one-domain-per-file; new domains get their own file plus a star-export row in `schema/index.ts`, not an append to `config.ts`.

Parent: [`../AGENTS.md`](../AGENTS.md)

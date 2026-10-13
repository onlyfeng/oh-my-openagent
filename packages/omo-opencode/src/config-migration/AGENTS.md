# src/config-migration/ - Legacy Config Discovery + Transforms

## OVERVIEW

**Updated:** 2026-10-13 / dc373d26f

Discovers legacy config files (oh-my-opencode/oh-my-openagent JSON[C], `~/.omo/config.jsonc`) and builds migration plans that transform them into the unified `omo.json[c]` chain. The migration *engine* (lock, journal, no-clobber merge, atomic commit) lives in `packages/omo-config-core/src/migration/`; this directory owns the OpenCode-side discovery and content transforms. Consumed at plugin startup (`src/startup-migration.ts` `runOpenCodeStartupMigration`, plus senpi config-startup), codex startup, `config migrate`, and the doctor deprecated-key / legacy-leftover checks. Exported as package subpath `@oh-my-opencode/omo-opencode/config-migration` (guarded by `config-migration-export.test.ts`). 18 implementation files, 11 tests.

## MIGRATION IDS

| Id | Constant | Transform |
|----|----------|-----------|
| `2026-07-opencode-config-unification` | `OPENCODE_CONFIG_MIGRATION_ID` (`discovery.ts`) | `transform-opencode.ts`: oh-my-* files into `omo.json[c]` |
| `2026-07-codex-config-jsonc` | `CONFIG_JSONC_MIGRATION_ID` (`discovery.ts`) | `transform-config-jsonc.ts`: `~/.omo/config.jsonc` |
| `2026-08-reasoning-unification` | `REASONING_UNIFICATION_MIGRATION_ID` | `reasoning-unification.ts`: `variant` / `reasoningEffort` / `thinking` / `textVerbosity` / `fallback_models` into `reasoning` + `models` |
| `2026-09-category-deep-split` | `CATEGORY_DEEP_SPLIT_MIGRATION_ID` | `category-deep-split.ts`: rename legacy category names to canonical ones (drops a legacy key when the canonical one is already set) |
| `2026-09-harness-native-rename` | `HARNESS_NATIVE_RENAME_MIGRATION_ID` | `harness-native-rename.ts`: canonicalize legacy harness block names |
| `2026-09-subscription-provider-rename` | `SUBSCRIPTION_PROVIDER_RENAME_MIGRATION_ID` | `subscription-provider-rename.ts`: legacy subscription provider ids (`hasLegacySubscriptionProviderIds`) |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Which files count as legacy | `discovery.ts` + `discovery-roots.ts` + `discovery-paths.ts` (ids `OPENCODE_CONFIG_MIGRATION_ID`, `CONFIG_JSONC_MIGRATION_ID`) |
| Plan building | `migration-plans.ts` (`createLegacyConfigMigrationPlans`) |
| Plan execution | `migration-executor.ts` (`executeLegacyConfigMigrationPlan`) |
| Content transforms | `transform-opencode.ts` / `transform-config-jsonc.ts` |
| Reasoning-key unification | `reasoning-unification.ts` (fixtures under `2026-08-reasoning-unification/`) |
| Rename transforms | `category-deep-split.ts`, `harness-native-rename.ts`, `subscription-provider-rename.ts` (fixtures under `2026-09-subscription-provider-rename/`) |
| Shared helpers | `record-values.ts` (`isPlainRecord`, `copyRecord`, `mergeRecords`, `withoutLegacyMetadata`), `schema-url.ts` (`OMO_SCHEMA_URL`), `transform-types.ts`, `types.ts` |
| Historical migration ids | `legacy-history.ts` |

## CONVENTIONS

- Transforms are pure: they take loaded sources and return `ConfigMigrationTransformResult` (`{ document, diagnostics }`, one diagnostic line per rename or drop); all filesystem side effects go through the omo-config-core engine.
- Every migration is identified by a stable id recorded in `_migrations`; never reuse or rename shipped ids (append to `legacy-history.ts` instead).
- `deep-diff.ts` backs no-clobber semantics: existing target values always win over migrated values.

## ANTI-PATTERNS

- Never write config files directly from here; hand plans to the engine (`runMigration`).
- Never delete legacy sources; migration copies forward and leaves timestamped backups.

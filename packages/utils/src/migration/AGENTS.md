# migration - Config Key Migration Maps

**Generated:** 2026-11-18 / c2691818a9

**Score:** 10 (10 files, ~1.1k LOC; distinct domain with public subpaths `./migration` + `./migration/*`). The parent [`../../AGENTS.md`](../../../AGENTS.md) names the maps; this file maps the files and the sidecar rule.

## WHERE TO LOOK

| Task | File |
|------|------|
| Agent-name renames | `agent-names.ts` |
| Hook-name renames | `hook-names.ts` |
| Model-version renames | `model-versions.ts` |
| Agent-category renames | `agent-category.ts` |
| Config-file migration flow | `config-migration.ts` - exports `migrateConfigFile(configPath, rawConfig): boolean` (returns needsWrite; only its colocated test consumes it). `../migration.ts` re-exports ONLY the four map modules, not this |
| Applied-migration state | `migrations-sidecar.ts` - sidecar state file tracking applied migrations OUTSIDE the user's config file, written atomically (`writeFileAtomically`) |

## CONVENTIONS

- One rename family per map file; tests colocated (`agent-names.test.ts`, `config-migration.test.ts`, `migrations-sidecar.test.ts`, `model-versions.test.ts`).
- Migration progress is durable in the sidecar, never inferred from the config file's current shape.
- Subpath consumers import `@oh-my-opencode/utils/migration` or `.../migration/<file>`, not relative paths.

## ANTI-PATTERNS

- Do not record applied migrations inside the user's config file.
- Do not add a rename without its map entry + test.

## GOTCHAS (source-verified)

- Sidecar format: `<configPath>.migrations.json` holding `{ appliedMigrations: [...] }`; keys like `model-version:old->new`, sorted, written atomically; an EMPTY set writes no sidecar file at all.
- Why the sidecar exists (#3263): users reverting auto-migrated values and deleting `_migrations` fell into an infinite re-migration loop every startup; reset requires deleting config AND sidecar.
- `migrateConfigFile` unions applied keys from sidecar + legacy in-config `_migrations`/`appliedMigrations`, strips the legacy fields from the config body, but keeps `_migrations` in the first write so a sidecar-write failure cannot strand migrated state off disk.
- Non-rename migrations live inline in `config-migration.ts`: `omo_agent` → `sisyphus_agent`; retired `lsp` key deleted (custom servers moved to `.opencode/lsp.json` for the lsp MCP); `experimental.hashline_edit` hoisted to top-level `hashline_edit`.
- `HOOK_NAME_MAP` values may be `null` - hook removed; `migrateHookNames` filters it from `disabled_hooks` and returns it in `removed`.
- `MODEL_VERSION_MAP` takes only genuinely retired models (currently just `anthropic/claude-opus-4-4`); `CURRENT_USER_SELECTABLE_MODELS` is an explicit guard - auto-rewriting a live user choice broke configs (#3777, #4527).
- `AGENT_NAME_MAP` collapses ~30 legacy variants (`"OmO-Plan"`, `"Planner-Sisyphus"`, ...) onto the 10 `BUILTIN_AGENT_NAMES`.

Parent: [`../AGENTS.md`](../AGENTS.md) (src layout) and [`../../AGENTS.md`](../../AGENTS.md) (category row).

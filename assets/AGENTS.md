# assets/ - Generated Schemas

## OVERVIEW

Checked-in JSON artifacts consumed by editors, CLIs, installers, and published packages. This directory is output, not the source of configuration or help contracts.

## STRUCTURE

```
assets/
├── oh-my-opencode.schema.json   # OpenCode edition config schema (script marks it LEGACY; still published)
├── omo.schema.json              # Harness-neutral omo.json / omo.jsonc schema
└── help/                        # Machine-readable CLI help result schemas
    ├── acp.schema.json          # ACP server status output
    ├── doctor.schema.json       # doctor diagnostic output
    ├── sandbox.schema.json      # sandbox environment output
    └── status.schema.json       # system status output
```

## SOURCE OF TRUTH

Map by GENERATOR, not by file — one command writes several artifacts:

| Generator | Zod source | Writes |
|-----------|------------|--------|
| `bun run build:schema` (`script/build-schema.ts`) | `packages/omo-config-core/src/schema` + `packages/omo-opencode/src/config/schema` | `assets/omo.schema.json`, `assets/oh-my-opencode.schema.json`, `dist/oh-my-opencode.schema.json` |
| `bun run build:omo-schema` (`script/build-omo-schema.ts`) | omo document only | `assets/omo.schema.json` |
| `bun run script/build-help-schemas.ts` | `packages/omo-opencode/src/help/schema/{doctor,status,sandbox,acp}.ts` | all four `assets/help/*.schema.json` |

- `dist/oh-my-opencode.schema.json` is the published package export (package.json `"./schema.json"`); it regenerates ONLY via `build:schema`.
- All artifacts are draft-07 with `$id` pinned to raw.githubusercontent.com dev-branch URLs.

## COMMANDS

```bash
bun run build:schema                   # umbrella: both assets schemas + dist copy
bun run build:omo-schema               # omo.schema.json only
bun run script/build-help-schemas.ts   # assets/help/*.schema.json
bun test tests/omo-schema-freshness.test.ts
```

## CONVENTIONS

- The freshness gate (`tests/omo-schema-freshness.test.ts`) covers ONLY `omo.schema.json`; the legacy and help schemas rely on the commit-with-source-change convention.
- Regenerate after changing the corresponding source schema or help definition.
- Commit generated output with the source change that requires it.
- Review diffs for accidental field removal, default changes, or stale descriptions.

## ANTI-PATTERNS

- Never hand-edit generated JSON.
- Never fix a freshness test by weakening or deleting the assertion.
- Never update only one schema copy when a shared source feeds multiple artifacts (`build:schema` alone writes three).

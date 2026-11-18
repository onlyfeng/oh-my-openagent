# scripts/ - Root Node Helpers

## OVERVIEW

Small Node ESM helpers that are intentionally separate from the primary Bun/TypeScript automation in singular `script/`.

## WHERE TO LOOK

| File | Purpose |
|------|---------|
| `check-third-party-notices.mjs` | Validate third-party notice coverage for source and shipped payloads; `parseNpmPackJson` normalizes npm 11 (array) and npm 12 (keyed object) `npm pack --dry-run --json` manifests |
| `third-party-notice-requirements.mjs` | Declarative notice requirements and path/package matching |
| `check-third-party-notices.test.mjs` | Node test coverage for spawn invocation resolution (Windows/non-Windows) and `parseNpmPackJson` (npm 11 array, npm 12 keyed, malformed and ambiguous rejection) |
| `desktop-package-boundaries.test.mjs` | Architecture test: allowlist of workspace edges for every `packages/senpi-desktop-*` package, host-package ban, SDK builtin ban |
| `knip.workspace.jsonc` | Knip fragment for this directory; declares the notice checker as a root-workspace entry. Composed by `script/knip-config.ts` |

## COMMANDS

```bash
node --test scripts/check-third-party-notices.test.mjs
node --test scripts/desktop-package-boundaries.test.mjs
node scripts/check-third-party-notices.mjs
node scripts/check-third-party-notices.mjs --ship
```

`bun run test:codex` invokes the `--ship` check after building the Codex payload. `bun run test:desktop` runs the desktop boundary test. Release workflows also rely on this surface before publication.

The checker test covers spawn invocation resolution (Windows/non-Windows) and `parseNpmPackJson` manifest parsing (npm 11/12 formats, malformed and ambiguous payloads); it does not cover notice discovery or failure cases. The default checker command is not a claim that the checker currently passes.

## CONVENTIONS

- Keep these files plain Node ESM; they run in packaging and compatibility flows that should not depend on Bun APIs.
- Put general build, release, QA, and repository-invariant automation in `script/`, not here.
- Add notice requirements declaratively in `third-party-notice-requirements.mjs`; keep traversal and reporting in the checker.
- Error output must identify the missing requirement or unexpected shipped path without exposing credentials or environment dumps.
- The boundary test enumerates `packages/senpi-desktop-*` and fails until a new package has an `ALLOWED_EDGES` row; add the row in the same change that adds the package.
- Knip entries for this directory's files go in `knip.workspace.jsonc` here, not in root `knip.ts`.

## ANTI-PATTERNS

- Do not move these helpers into `script/` without updating package scripts, CI, and published-path assumptions.
- Do not skip the `--ship` mode when changing the Codex or npm payload.
- Do not duplicate third-party package rules inside tests; test the exported requirement table.
- Do not loosen `ALLOWED_EDGES`, `FORBIDDEN_HOST_PACKAGES` (`pi-agent-core`, `pi-ai`, `pi-tui`), or the SDK builtin ban to make a violation disappear. The only sanctioned builtin exception is `node:net` in `packages/senpi-desktop-sdk/src/transport/attach-uds.ts`.

# get-worker - get.omo.dev Cloudflare Worker

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Cloudflare Worker `omo-get` (package `omo-get-worker`, unscoped, private) behind the `get.omo.dev` custom domain: serves the OmO Native installer scripts, release channel pointers, mirrored release assets, and download stats. Earned its own file as a distinct domain (score ~12): Worker runtime, own `bun.lock`, `wrangler.jsonc`, D1 migrations, and tests outside `src/`. No other package imports it.

## ROUTES (`src/index.ts` `route()`; GET/HEAD only, else 405)

| Path | Handler | Behavior |
|------|---------|----------|
| `/` | `scripts.ts` `scriptForRoot` | `curl`/`Wget`/`fetch`/`HTTPie` UA gets `install.sh`, PowerShell UA gets `install.ps1`, anything else 302s to the docs page |
| `/install.sh`, `/install.ps1` | `serveScript` | `text/plain` + `nosniff`, 300 s cache |
| `/channels/<latest\|beta>` | `channels.ts` `serveChannel` | R2 pointer first (moves only after a verified mirror), npm `omo-ai` dist-tag fallback |
| `/v/<version>/<asset>` | `serve-release.ts` `serveReleaseAsset` | edge cache, then R2 (only when the version's completion marker exists), else 302 to the GitHub release asset with `X-Omo-Fallback-Reason` |
| `/stats/downloads` | `download-stats.ts` | D1 rollup read, fetched by the `web` package |
| `/healthz` | inline | `ok` |

`scheduled()` runs `downloads-rollup.ts` `runDownloadsRollup` hourly at :17 (Analytics Engine SQL query to D1, needs the `ANALYTICS_TOKEN` secret). `datapoint.ts` `recordDownload` writes one Analytics Engine point per GET (`isQaInstall` flags QA traffic). `release-names.ts` owns the version regex, the asset allowlist (`omo-*` binaries, `SHA256SUMS`, `senpi-desktop-engine-*`), and the R2/GitHub key shapes.

## BINDINGS (`wrangler.jsonc`)

`RELEASES` (R2 `omo-releases`), `DB` (D1 `omo-get`, `migrations/`), `DOWNLOADS` (Analytics Engine `omo_downloads`), vars `ACCOUNT_ID`, `ANALYTICS_DATASET`, `ROLLUP_FIRST_DAY`. `Env` / `RequestContext` types live in `src/env.ts`. Regenerate binding types with `bun run cf-typegen`.

## CONVENTIONS

- `.sh` / `.ps1` import as strings: `bunfig.toml` `[loader]` for tests, wrangler `rules` `Text` for the bundle, `src/text-modules.d.ts` for types.
- tsconfig is stricter than the repo norm (`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `verbatimModuleSyntax`, `noUnused*`, types `workers-types` + `bun`).
- Tests live in `test/` (not co-located) and use `test/fakes.ts` for R2/D1/cache/Analytics fakes.
- `scripts/install.sh` is a POSIX-sh wrapper whose Bash program is a quoted heredoc written to a temp file and run from `main()` on the last line, so a truncated download never runs half a script. `install.ps1` is its PowerShell twin. Env knobs: `OMO_INSTALL_DIR`, `OMO_NO_MODIFY_PATH`, `OMO_INSTALL_BASE_URL`, `OMO_INSTALL_ALLOW_SUDO`.

## ANTI-PATTERNS

- Never deploy with a pending D1 migration: `scripts/check-migrations-applied.sh` fails closed when a `migrations/*.sql` is missing from `migrations/applied.txt` (the deploy token has no D1 access; migrations go through the reviewed infra path first).
- Never let the hourly rollup write days before `ROLLUP_FIRST_DAY` (`rowsFrom` filters them; earlier days were backfilled across the account move).
- Never serve an R2 object for a version without its completion marker; fall back to GitHub instead.
- Never hand-deploy; `get-worker-deploy.yml` (environment `omo-production`) owns deploys.

## COMMANDS

```bash
# from packages/get-worker/
bun install --frozen-lockfile
bun run type-check        # tsgo --noEmit
bun run test              # bun test (test/)
bun run lint:scripts      # shellcheck scripts/install.sh
bunx wrangler deploy --dry-run --outdir dist   # CI bundle check
bun run dev               # wrangler dev
```

CI: `get-worker-ci.yml` (unit + shellcheck + PSScriptAnalyzer + install smokes on Linux glibc/arm64, macOS, Alpine musl, Windows). Parent: [`packages/AGENTS.md`](../AGENTS.md).

# get-worker — `get.omo.dev` Cloudflare Worker

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 12 (29 files, ~1.3k LOC, standalone package boundary; role in [`../AGENTS.md`](../AGENTS.md) ROLE MAP)

## OVERVIEW

`omo-get-worker` (private): the install-script and release-serving Worker behind `get.omo.dev`. Own `bun.lock`, `wrangler.jsonc`, D1 migrations, and CI/deploy workflows. GET/HEAD only; everything else is 405.

## ROUTES (`src/index.ts` `route()`)

| Route | Handler | Notes |
|-------|---------|-------|
| `/` | `scripts.ts` `scriptForRoot(request)` | Install script chosen by User-Agent; `docsRedirect()` when no script matches |
| `/healthz` | inline | `ok` |
| `/stats/downloads` | `download-stats.ts` | Serves the rollup table |
| `/<script-name>` | `scripts.ts` | Named install scripts |
| `/channels/:channel` | `channels.ts` | Unknown channel → 404 |
| `/v/:name/:asset` | `serve-release.ts` | Release assets from R2 |

## BINDINGS (`wrangler.jsonc`)

- **R2** `RELEASES` → bucket `omo-releases`.
- **D1** `DB` → `omo-get` (`migrations/0001_downloads.sql`, `applied.txt`; `scripts/check-migrations-applied.sh` guards deploys).
- **Analytics Engine** `DOWNLOADS` → `omo_downloads` (per-request download counting).
- **Cron** `17 * * * *` → `downloads-rollup.ts` hourly rollup; needs the read-only `ANALYTICS_TOKEN` secret. `ROLLUP_FIRST_DAY` (2026-10-10) marks the day the account moved: the rollup must never overwrite it — a one-time backfill (both accounts' Analytics, summed) completed it.
- Custom domain `get.omo.dev` on the omo.dev zone (a Worker reaches another Worker on the same zone via custom domain, not a route; `/stable/*`, `/beta/*` are omo-desktop-releases' zone routes). Text modules rule for `**/*.sh` / `**/*.ps1`.

## WHERE TO LOOK

| File | Role |
|------|------|
| `src/{channels,datapoint,download-stats,downloads-rollup,env,release-names,scripts,serve-release}.ts` | Route handlers + shared env/release-name logic |
| `scripts/install.sh`, `scripts/install.ps1` | Served installers; `lint:scripts` shellchecks install.sh |
| `test/` | `routes.test.ts`, `install-sh-*.test.ts` (preamble/detect-asset/other-installs), `migrations-applied.test.ts`, `downloads.test.ts`, shared `fakes.ts` |
| `knip.workspace.jsonc`, `bunfig.toml` | Workspace-scoped knip + test config |

## ANTI-PATTERNS

- Never deploy with unapplied D1 migrations (`migrations-applied` gate).
- Never let the hourly rollup overwrite `ROLLUP_FIRST_DAY`.
- No POST/PUT surfaces: the Worker is read-only by design.

## COMMANDS

```bash
bun run --cwd packages/get-worker dev          # wrangler dev
bun run --cwd packages/get-worker test         # bun test
bun run --cwd packages/get-worker type-check   # tsgo --noEmit
bun run --cwd packages/get-worker lint:scripts # shellcheck scripts/install.sh
```

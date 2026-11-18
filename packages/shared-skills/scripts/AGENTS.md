# shared-skills/scripts - Skill Materialization + Parity Dev Scripts

**Generated:** 2026-11-18 | c2691818a9

Build-time scripts that materialize the frontend third-party references out of pinned submodules and a parity detector for the upstream `impeccable` rule set. Development-only - nothing here ships in the npm tarball.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `frontend-refs-manifest.mjs` (+ `.d.mts`) | Single source of truth: which upstream paths land at which `skills/frontend/references/...` location. |
| `materialize-frontend-refs.mjs` | Path-mapped copy + `SKILL.md` `description:` quoting normalization; invoked by `omo-codex/plugin/scripts/materialize-shared-upstreams.mjs` on every ship path. |
| `designpowers-refs-manifest.mjs` | Manifest for the `designpowers` upstream. |
| `detect-source-parity.ts` | Diff local rules against an upstream checkout. |
| `detect-parity.ts` / `detect-parity-oracle.ts` | Run rules in a real browser (chrome/webkit) and emit a JSON verdict + oracle. |

## CONVENTIONS

- NEVER hand-edit a materialized copy under `skills/frontend/references/` - regenerate via `materialize-frontend-refs.mjs`. The materialized files are gitignored and re-ignored away in `.npmignore` so they ship.
- Parity scripts require a local upstream checkout (and a browser for the live oracle); they are not run in CI.

## COMMANDS

```bash
bun packages/shared-skills/scripts/detect-parity.ts <impeccable-checkout> [--rules id,id] [--backend chrome|webkit] [--json out.json] [--oracle browser.json]
bun packages/shared-skills/scripts/detect-parity-oracle.ts <impeccable-checkout> [--json out.json] [--jobs N]
bun packages/shared-skills/scripts/detect-source-parity.ts <upstream-checkout>
```

Parent: [`../AGENTS.md`](../AGENTS.md) - full frontend third-party pipeline + provenance gates.

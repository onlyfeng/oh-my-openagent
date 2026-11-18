# runtime — Tracked SDKs + Staged Vendored Runtimes

**Generated:** 2026-11-18 / c2691818a9

**Score:** 9 (6 subdirs; the tracked-vs-staged split is the trap this file exists for). Per-artifact detail lives in the parent [`plugin/AGENTS.md`](../AGENTS.md); this file is the ownership map.

## WHERE TO LOOK

| Need | Path |
|------|------|
| DAG orchestration (define/start/attach/snapshot/wait/cancel/retry/send/amend) | `dag/sdk.js` |
| Named/reusable dag definitions (the `dag-library` skill's save target) | `dag/library.js` |
| `omo thread` CLI thread SDK (runs the gateway store worker) | `thread-sdk/sdk.js` |
| Agent-toolkit eval SDK (qa lanes) | `agent-toolkit-sdk/sdk.js` |
| bunshin-ts fleet SDK | `bunshin-ts/sdk.js` |
| Rollback migration of persisted task state (host-session sockets, status/cost fields) | `rollback-migrate.js` |

## TRACKED (git-visible, ship with the plugin)

`git ls-files` under this dir returns exactly: `agent-toolkit-sdk/sdk.js`, `bunshin-ts/sdk.js`, `dag/library.js`, `dag/sdk.js`, `rollback-migrate.js`, `thread-sdk/sdk.js` — all generated (fingerprint markers prove freshness), never hand-edited. `rollback-migrate.js` is minified with a `// omo:<fingerprint>` marker; its source of truth is `../src/extension/rollback-migrate-runtime.ts`, and the omo-native launcher loads it as a subprocess runtime. `bunshin-ts/sdk.js` is vendored from `../vendor/` with `#bunshin-ts-dep/*`-pinned imports.

## STAGED (gitignored, vendored at build time)

`ast-grep-mcp/` (staged from `packages/ast-grep-mcp` `dist/cli.js` by `plugin/scripts/stage-ast-grep-mcp-runtime.mjs`) and `lsp-daemon/` — absent in a fresh checkout until the plugin build stages them.

## ANTI-PATTERNS

- NEVER hand-edit anything here — tracked SDKs are generated (fingerprint markers prove freshness), staged dirs are build outputs. Regenerate via `plugin/scripts/`.
- Treat minified bundles as opaque: LSP diagnostics on them are noise; source of truth is `../src/` plus the build scripts.
- Do not commit anything new here without a build-script change that generates it.

## COMMANDS

```bash
node plugin/scripts/build-extension.mjs --check   # fingerprint freshness
bun run test:senpi                                 # repo gate: build + stage + typecheck + test
```

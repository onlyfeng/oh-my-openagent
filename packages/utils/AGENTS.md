# utils - Shared Utilities (Core)

**Generated:** 2026-10-13 / dc373d26f (prior f3642fcda)

## OVERVIEW

Harness-neutral pure-TypeScript core package (`@oh-my-opencode/utils`). Consumed by both adapters (`packages/omo-opencode`, `packages/omo-codex`) and the other Core packages. Root barrel `src/index.ts` re-exports 34 modules (runtime shims, config tooling, file utilities, prompt gating, git parsing, migration maps); `process-sweep` is NOT in the root barrel, import it via the `./process-sweep` subpath. `package.json` `exports` also publishes per-module subpaths (`./runtime`, `./prompt-async-gate/*`, `./migration/*`, `./git-worktree/*`, `./command-executor/*`, `./zip-entry-listing/*`, ...).

## CATEGORY MAP

| Group | Key Files | Role |
|-------|-----------|------|
| **Deep Merge** | `deep-merge.ts` | `deepMerge()`: recursive, prototype-pollution safe (`__proto__`/`constructor`/`prototype` filtered), max depth 50 |
| **Config** | `config-merge.ts`, `config-section-parser.ts`, `env-expansion.ts` | Unique-string list merges (`mergeUniqueStrings*`), per-section schema parse with issue paths (unsafe keys rejected), `${VAR}` expansion with allowlist blocking |
| **Frontmatter** | `frontmatter.ts` | `parseFrontmatter()`: default YAML (`js-yaml` JSON_SCHEMA) + rule-mode parser with multiline glob arrays |
| **File Utils** | `file-utils.ts`, `atomic-write.ts`, `write-file-atomically.ts`, `xdg-data-dir.ts`, `contains-path.ts`, `archive-entry-validator.ts`, `classify-path-environment.ts`, `skill-path-resolver.ts` | Symlink resolution, atomic writes with tolerant fsync, XDG data dir with tmp fallback, path containment, archive entry validation |
| **Runtime Shims** | `runtime/spawn.ts`, `runtime/which.ts`, `runtime/file.ts` + Git Bash resolution | `spawn()` / `spawnSync()`: prefers `Bun.spawn`, falls back to Node `child_process`; unified `SpawnedProcess` interface; `bunWhich`/`bunFile`/`bunWrite` |
| **Prompt Gate** | `prompt-async-gate.ts` + `prompt-async-gate/*.ts` (13 files), `prompt-failure-classifier.ts`, `session-idle-settle.ts`, `internal-initiator-marker.ts` | `dispatchInternalPrompt()`: reservation + queue + semantic dedupe + pending-tool-turn + live-route fallback; the mandated gate for all internal `session.prompt` calls. Owner regression: `src/prompt-async-gate.test.ts` (+ `prompt-async-gate-*.test.ts` siblings) |
| **Tool Args** | `tool-name.ts`, `replace-tool-args.ts`, `shell-command-escape.ts` | Tool-name normalization, arg rewriting (audited by `replace-tool-args.audit.test.ts`), shell escaping |
| **Git Worktree** | `git-worktree/*.ts` | Porcelain status parser, diff stat collection, `formatFileChanges()` |
| **Migration** | `migration.ts`, `migration/*.ts` | Agent-name, hook-name, model-version, agent-category migration maps; `migrateConfigFile()` |
| **Command Exec** | `command-executor/execute-command.ts`, `execute-hook-command.ts` | `executeCommand()` via `node:child_process` `exec`; stderr inlined in return string |
| **Ast-grep** | `ast-grep/*.ts` | sg binary provisioning: pinned manifest, resolver/cache, install script; `provisionSgBinary()` |
| **Process Sweep** | `process-sweep/*.ts` | Family-based orphan cleanup: LSP daemon proxy/version sweeps, ownership attestation |
| **Process Tree** | `process-tree.ts`, `process-tree-termination.ts`, `process-stream-reader.ts` | Process tree enumeration/termination, stream reading |
| **Logging** | `logging/*.ts` | Buffered logger (flush/rotation) + product identity |
| **Zip Listing** | `zip-entry-listing/*.ts` | tar / zipinfo / Python / PowerShell adapters + symlink target reading |
| **Ports** | `port-utils.ts` | `findAvailablePort()`, `DEFAULT_SERVER_PORT` |
| **Misc** | `snake-case.ts`, `jsonc-parser.ts`, `record-type-guard.ts`, `logger.ts`, `extract-semver.ts`, `format-duration.ts` | Key transformation, memoized safe JSONC parse, type guards, injectable shared logger stub |

## NOTES

- **Dependencies:** `js-yaml`, `jsonc-parser`, plus a declared `@oh-my-opencode/omo-config-core` workspace dep (no `src/` import of it today).
- **Commands:** `bun run --cwd packages/utils test` (`bun test src/*.test.ts src/**/*.test.ts`), `bun run --cwd packages/utils typecheck`.
- **Legacy maps:** `migration/agent-category.ts` is DEPRECATED, DO NOT add entries; `migration/model-versions.ts` must not map current user-selectable model variants.
- **sg provisioning:** a candidate binary is accepted only if its version probe runs and reports `ast-grep`; provisioning verifies SHA-256 and rejects writes outside the target dir; the install-script timeout MUST stay ref'd (unref hung the Windows runner).
- **No path aliases:** relative imports only; strict `ESNext` + `bundler` moduleResolution.
- **Runtime portability:** `runtime/spawn.ts` wraps both Bun and Node APIs so downstream packages can run under Electron-hosted OpenCode.
- **Side effects:** `package.json` `sideEffects` pins `src/prompt-async-gate/queue.ts`; tree-shaking must keep the queue module.
- **Process-sweep invariant:** the LSP daemon *server* shape (`cli.js daemon`, no `mcp`) is NEVER a proxy candidate — only the proxy shape is swept (`process-sweep/lsp-proxy-family.ts`, pinned by tests).
- **Prompt-async-gate is critical infrastructure:** every internal prompt dispatch across the plugin must route through `dispatchInternalPrompt()` to prevent duplicate injections and race conditions. See the root [`AGENTS.md`](../../AGENTS.md) "Internal message injection is dangerous" section.

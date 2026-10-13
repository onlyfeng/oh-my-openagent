# lsp-core -- Harness-Neutral LSP Engine (Core)

**Generated:** 2026-10-13 / dc373d26f

## OVERVIEW

Harness-neutral LSP engine (`@oh-my-opencode/lsp-core`, only workspace dep `mcp-stdio-core`, source-only, no build). Manages language server lifecycle, JSON-RPC transport, configuration merging, and tool definitions. Consumed by the MCP-layer packages [`lsp-tools-mcp`](../lsp-tools-mcp) and [`lsp-daemon`](../lsp-daemon/AGENTS.md), and directly by `omo-senpi` (`src/components/lsp/`) and `omo-codex`. See parent [packages/AGENTS.md](../AGENTS.md).

## KEY FILES

| File | Role |
|------|------|
| `src/lsp/manager.ts` | `LspManager`: ref-counted client pool (`MAX_RESIDENT_CLIENTS` = 6), init timeout, idle reaper, per-key respawn budget, abort signals |
| `src/lsp/client.ts` | `LspClient`: `openFile`, `definition`, `references`, `symbols`, `diagnostics`, `rename` |
| `src/lsp/client-wrapper.ts` | `withLspClient()`: workspace root discovery, retry on dead connection, release |
| `src/lsp/connection.ts` | `LspClientConnection`: `initialize` request with capabilities, settle delay |
| `src/lsp/json-rpc-connection.ts` | Raw JSON-RPC 2.0 framing over stdio |
| `src/lsp/config-loader.ts` | Load `.codex/lsp-client.json` (project + user), merge with builtins |
| `src/lsp/server-definitions.ts` | `BUILTIN_SERVERS` (42 entries), `LSP_INSTALL_HINTS` (42), `AUTO_INSTALLABLE_SERVERS` (21) |
| `src/lsp/server-resolution.ts` | `findServerForExtension()`: map extension to installed server; substitutes the resolved absolute binary path into `command[0]` |
| `src/lsp/server-installation.ts` | `resolveServerBinary()`: marker-gated repo-local lookup then PATH, with Windows extension handling; `isServerInstalled()` retained for existing consumers |
| `src/lsp/directory-diagnostics.ts` | `aggregateDiagnosticsForDirectory()`: walk directory, cap files + diagnostics; `AbortSignal` cancels acquisition and per-file scans |
| `src/lsp/formatters.ts` | Format locations, symbols, diagnostics, rename results, workspace edits |
| `src/lsp/workspace-edit.ts` + `workspace-edit-*.ts` | `applyWorkspaceEdit()` / `applyWorkspaceEditDetailed()`: parse → fingerprint/snapshot → simulate → commit pipeline |
| `src/lsp/workspace-mutation-controller.ts` | Lease/concurrency validation for workspace filesystem mutations |
| `src/lsp/fixtures/` | Test-only LSP servers/probes (workspace-edit server, diagnostics-freshness contract probe) |
| `src/lsp/constants.ts` | Limits and timings (`REQUEST_TIMEOUT_MS` 15 s, `INIT_TIMEOUT_MS` 60 s, `IDLE_TIMEOUT_MS` 5 min, respawn limit/cooldown); `timer-provider.ts` is the injectable clock seam |
| `src/post-edit/orchestration.ts` | Post-edit diagnostics blocks (capped concurrency, not-configured cache) via `./post-edit` subpath; `file-location.ts` never treats the home dir as a project root |
| `src/missing-dependency-result.ts` | Shared missing-dependency MCP result shape via `./missing-dependency-result` subpath |
| `src/tools/definitions.ts` | `LSP_MCP_TOOLS`: 9 tool schemas exported to MCP; one executor module per tool family (`diagnostics`, `navigation`, `symbols`, `rename`, `format`, `status`, `install-decision`) |
| `src/tools/runtime.ts` | `executeLspTool()` + `coerceToolArguments()` dispatch |
| `src/request-context.ts` | `runWithRequestContext()` / `contextCwd()` / `contextEnv()` via `AsyncLocalStorage` |
| `src/mcp.ts` | `handleLspMcpRequest()` + `runMcpStdioServer()`: MCP entry over `mcp-stdio-core` |

## NOTES

- **Tool surface:** 9 tools, each named bare (`status`, `format`, ...) with an `lsp_*` alias: `lsp_status`, `lsp_diagnostics`, `lsp_goto_definition`, `lsp_find_references`, `lsp_symbols`, `lsp_prepare_rename`, `lsp_rename`, `lsp_format` (writes the server's formatting edits to disk), and `lsp_install_decision`. Pinned by `src/tool-surface.test.ts` (+ `mcp-protocol-pin.test.ts`).
- **Subpath exports:** `.`, `./tools`, `./request-context`, `./missing-dependency-result`, `./mcp`, `./post-edit`, and wildcard `./lsp/*` — all point at source `.ts`, no dist build.
- **RequestContext seam:** `request-context.ts` uses `AsyncLocalStorage` so the MCP proxy can thread `cwd` and `env` through shared daemon sessions.
- **Config priority:** project `.codex/lsp-client.json` beats user `~/.codex/lsp-client.json` beats `BUILTIN_SERVERS`.
- **Binary resolution:** `resolveServerBinary()` probes a path-shaped command, then marker-gated repo-local bin directories walking up from the request cwd, then `PATH`. A local bin directory is only trusted when a sibling project marker (`package.json`, `pyproject.toml`, `Gemfile`, `go.mod`, ...) proves it belongs to that ecosystem, so an unmarked `node_modules/.bin` is never trusted; the walk stops at a `.git` boundary. Results are cached per process, keyed by working directory and command. Live QA driver: `scripts/qa/local-binary-resolution-e2e.mjs`.
- **Reaper:** `LspManager` reaps clients idle longer than `IDLE_TIMEOUT_MS` (default 5 min) or stuck initializing past `INIT_TIMEOUT_MS` (default 60 s). After `CLIENT_RESPAWN_RETRY_LIMIT` (2) dead generations for one key, admission throws `LspClientRespawnBudgetExceededError` until `CLIENT_RESPAWN_COOLDOWN_MS` (60 s) passes.
- **Tests:** bun:test, `#given/#when/#then`; heavy invariants live in `*.integration.test.ts` driven by `src/lsp/fixtures/` servers, plus characterization and adversarial workspace-edit suites. Run `bun test src/*.test.ts src/**/*.test.ts`.
- **Workspace-edit safety:** paths are canonicalized, prevalidated, simulated, and lease-guarded before commit; client wrappers reject paths outside the request context (`outside-context-workspace.ts`).
- **Directory diagnostics cancellation:** `aggregateDiagnosticsForDirectory()` accepts an `AbortSignal`; it throws if aborted before acquisition, threads the signal into `manager.getClient()` so a cold acquisition is cancellable, and checks it between files. An aborted cold start rejects with `AbortError` and leaves `clientCount()` at 0.

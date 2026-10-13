# lsp component

## Overview

Daemon-backed Senpi LSP adapter. This component retains only Senpi-facing descriptors, schemas, renderers, path extraction, post-edit wiring, and migration-warning helpers adapted from `pi-lsp-client` / oh-my-pi by Yeongyu Kim. Client, transport, JSON-RPC, manager, server-resolution, project-trust, diagnostics, workspace-edit, and process execution live in `@oh-my-opencode/lsp-core` plus the packaged `@code-yeongyu/lsp-daemon` runtime. Every tool call is forwarded out of process: the component never spawns language servers itself.

## Structure

| Path | Purpose |
|------|---------|
| `index.ts` | Component factory (`createLspComponent`): flag registration, six-tool registration with the daemon runtime wrapped over each `execute`, the `tool_result` hook (format-on-mutation, then post-edit diagnostics), session lifecycle (`session_start`/`session_compact`/`session_shutdown` state reset plus daemon-client cache clear). |
| `daemon-runtime.ts` | Resolves the packaged daemon CLI at `../runtime/lsp-daemon/dist/cli.js` (path must exist, version validated against a strict pattern); `OMO_LSP_DAEMON_CLI` + `OMO_LSP_DAEMON_VERSION` override both-or-neither, absolute path only. |
| `daemon-tool-client.ts` | Loads the daemon's `client.js` beside the CLI (cached per path), maps Senpi tool names to daemon names (`lsp_diagnostics` -> `diagnostics`, ...), and builds the per-call request context: canonical cwd, the project config paths `.omo/lsp-client.json` then `.pi/lsp-client.json`, the user config path, the install-decisions path, `installDecisionTool: false`. All paths come from `config-paths.ts`. |
| `config-paths.ts` | Where LSP config lives: the branded `.omo/` directory is read first and `.pi/` stays a read-only fallback for trees the brand-dir migration (#8370) has not moved yet. A user file (`lsp-client.json`, `lsp-install-decisions.json`) resolves to the first existing of `~/.omo/` and `~/.pi/`, defaulting to `~/.omo/` when neither exists. |
| `post-edit-diagnostics.ts` | Post-edit diagnostics over `@oh-my-opencode/lsp-core/post-edit`: mutation tool names and path extraction come from `../post-mutation/post-mutation.ts` (shared with `formatter`), concurrency 4, per-session not-configured cache, widget lines under key `omo-senpi-lsp`. |
| `post-edit-outcome.ts` | `postEditOutcomeFromDaemonResult`: joins daemon text blocks and maps a `not_configured` availability detail to the lsp-core outcome. |
| `adapter/descriptors.ts` | The six ToolDefinitions (`lsp_diagnostics`, `lsp_goto_definition`, `lsp_find_references`, `lsp_symbols`, `lsp_prepare_rename`, `lsp_rename`) with typed details and TUI renderers. |
| `adapter/renderers-*.ts` / `renderers.ts` / `rendering.ts` / `render-format.ts` | Call/result renderers per tool family (diagnostics, navigation, rename, symbols); `renderers.ts` is the re-export barrel, `renderers-common.ts` the shared formatting. |
| `adapter/migration-notices.ts` | Scans the project's LSP config (the first existing of `.omo/lsp-client.json` and `.pi/lsp-client.json`) for non-disabled entries carrying `command`/`env` and produces `untrusted_project_lsp_command` notices. |
| `adapter/schema.ts` / `language-mappings.ts` | TypeBox `defineTool` helper and extension-to-language tables. |
| colocated tests | Component wiring, adapter renderers, the security contract (`lsp-security.test.ts`), daemon runtime resolution, post-edit behavior (`post-edit-diagnostics.test.ts`, `post-edit-not-installed.test.ts`), formatter content (`formatter-content.test.ts`), and `lsp-architecture.test.ts`, which allows only the retained top-level files plus `adapter/` and forbids vendored JSON-RPC and project-command trust tokens. |

## Wiring

- `registerLspFlags` always runs; two boolean flags, both default `true`: `omo-senpi-lsp-tools-enabled` gates the whole component (early return when `false`), `omo-senpi-lsp-post-edit-diagnostics-enabled` gates only the post-edit hook.
- Each registered tool's `execute` ignores the Senpi-provided context and calls `callPackagedDaemonTool(name, args, {signal})`, which lazily imports the packaged daemon client and forwards with the current request context.
- Format-on-mutation runs first on every `tool_result`: a lazily built `createFormatterStep` (`../formatter/formatter.ts`, `formatOnMutation` from `loadSenpiOmoConfig`, project markers) appends its content; a `required`-mode failure returns `isError`. It runs even when post-edit diagnostics are off.
- Post-edit: on `tool_result` for a mutation tool, `handlePostEditDiagnosticsToolResult` reports "(OmO) Checking LSP Diagnostics" tool-hook status, runs `lsp_diagnostics` with `severity: "error"` through the daemon, appends diagnostic blocks to the tool result content, and syncs the `omo-senpi-lsp` widget. A daemon `not_configured` availability answer becomes a cached per-extension skip.
- `session_shutdown` deletes per-session post-edit state and clears the daemon client module cache; `session_compact` resets only the session's not-configured cache.

## Conventions

- Config-notice warnings go through `ctx.logger.warn`, deferred until after the first paint (`deferUntilAfterFirstPaint`) so the config reads never delay startup.
- The daemon version and CLI override are validated fail-closed: mismatched override pair, relative path, or missing file all throw at resolution.
- Session ids come from `sessionManager.getSessionId()` on the event context; a missing id degrades to an anonymous state slot, never a crash.

## Anti-patterns

- Do not read project-local `lsp-client.json` commands (`.omo/` or `.pi/`). They're intentionally ignored here; users who still need custom LSP commands must move those definitions to their user `~/.omo/lsp-client.json` (`~/.pi/lsp-client.json` is still read as a fallback). Project configs may keep safe fields such as extensions and priorities, but command/env entries only produce a migration warning.
- Do not add LSP client, transport, or server-management logic in this component; that belongs to `lsp-core` and the daemon package.
- Do not enable `installDecisionTool` in the request context; the Senpi adapter has no interactive install-decision surface.

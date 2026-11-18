# lsp — LSP Engine Internals (direct files)

**Score:** 17 (81 direct files, ~400KB; key-file roles are tabled in the parent [`packages/lsp-core/AGENTS.md`](../../AGENTS.md) — this file maps the CLUSTERS and test taxonomy instead of repeating that table). `fixtures/` is the only nested dir and is test-only.

## OVERVIEW

The engine itself: JSON-RPC transport, client/connection lifecycle, the manager pool, config/server resolution, the workspace-edit pipeline, diagnostics, formatting, and language mapping.

## CLUSTERS

| Cluster | Files (verified) |
|---------|------------------|
| Transport / protocol boundary | `json-rpc-connection.ts` (+ cancellation test), `connection.ts`, `errors.ts`, `constants.ts` |
| Client + wrapper | `client.ts`, `client-wrapper.ts` (workspace root discovery, retry on dead connection; `client-wrapper-outside-cwd.test.ts` pins context confinement) |
| Manager lifecycle | `manager.ts`, `manager-lifecycle.test.ts` (pooling, init timeout, idle reaper, tombstone stops, respawn budgets) |
| Config + servers | `config-loader.ts`, `server-definitions.ts`, `server-resolution.ts`, `server-installation.ts` |
| Workspace-edit pipeline | `workspace-edit.ts`, `workspace-edit-plan.ts`, `workspace-edit-commit.ts`, `workspace-edit-simulation.ts` + parsers/snapshot/fingerprint/path modules, `workspace-mutation-controller.ts`, `outside-context-workspace.ts` |
| Diagnostics | `directory-diagnostics.ts` (cancellable via `AbortSignal`), the `client-diagnostics-*` suites |
| Formatting + language | `formatters.ts`, `format-document.ts`, `language-mappings.ts`, `effective-extension.ts`, `infer-extension.ts` |
| Bun/host integration | `bun-command-output.ts`, `bun-project-checks.ts`, `host-js-runtime.ts`, `cleanup-errors.ts` |
| Test support | `controlled-clock-test-support.ts`, `fake-bun-test-support.ts`, `fixtures/` (workspace-edit server, diagnostics-freshness probe) |

## TEST TAXONOMY

- `*.test.ts` — unit; `*.integration.test.ts` — real-process integration (diagnostics concurrency/freshness, JSON-RPC cancellation).
- Characterization/adversarial suites cluster around the workspace-edit pipeline and transport; run the nearest cluster when touching its files.

## ANTI-PATTERNS

- Do not add filesystem writes outside the workspace-edit pipeline's lease/snapshot/commit path (`workspace-mutation-controller.ts` guards it).
- Do not block cold client acquisition without threading the caller's `AbortSignal` (directory-diagnostics contract).

## COMMANDS

```bash
bun test packages/lsp-core/src/lsp/manager-lifecycle.test.ts   # focused example
bun run --cwd packages/lsp-core typecheck
```

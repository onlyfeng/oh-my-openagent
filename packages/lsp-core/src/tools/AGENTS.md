# src/tools/ - LSP MCP Tool Layer

**Generated:** 2026-11-18 / c2691818a9

Earned its file (score ~25: barrel, >10 exports, every LSP tool call funnels through here). 18 TS files. The parent [packages/lsp-core/AGENTS.md](../../../AGENTS.md) already maps `definitions.ts`, `runtime.ts` and the pinned 8-tool surface - this file covers the operation modules underneath, not a repeat of that.

## WHERE TO LOOK

| Task | File |
|------|------|
| Add a tool, change a schema or a `lsp_*` alias | `definitions.ts` - one block per tool: name, aliases, title, description, `objectSchema` input, executor binding |
| Name/alias dispatch, argument coercion | `runtime.ts` - `executeLspTool()`, `coerceToolArguments()` |
| File diagnostics | `diagnostics.ts` - `executeLspDiagnostics` |
| Directory diagnostics in a Bun project | `bun-directory-diagnostics.ts` - `runBunDirectoryDiagnostics()`: a TypeScript directory is typechecked with `bun check`, plus `bun lint` when the installed Bun ships it |
| goto_definition / find_references | `navigation.ts` |
| prepare_rename / rename | `rename.ts` |
| Document symbols / server+config listing | `symbols.ts` / `status.ts` |
| Format / install decision | `format.ts` / `install-decision.ts` |
| Coercion + validation primitives | `parameters.ts` (`isRecord`, ...) |
| Result shaping / schemas / contracts | `result.ts` (`TextContent`, `ToolExecutionResult`) / `schema.ts` (`objectSchema`) / `types.ts` |
| Package-facing API | `index.ts` barrel - downstream code imports the barrel, never an operation module |

## CONVENTIONS

- A new tool = one `definitions.ts` entry + an `executeLsp*` function + a barrel export; the 8-tool surface stays pinned by `src/tool-surface.test.ts`.
- Operation modules wrap `withLspClient` so lifecycle, missing-dependency and cancellation behavior stays shared - do not re-implement client handling per tool.

## TESTS

Colocated and runnable per directory: `bun test packages/lsp-core/src/tools` (`bun-directory-diagnostics`, `format`, `parameters` suites).

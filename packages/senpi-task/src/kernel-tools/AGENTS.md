# kernel-tools - Eval-Kernel Tool Contract

**Score:** 13 (10 files, 1k LOC, index barrel, structural-validation domain). Parent [`packages/senpi-task/AGENTS.md`](../../AGENTS.md): "Kernel tool contract, bindings, nested-host scope, transcript names and error codes for tools invoked inside the eval kernel."

## OVERVIEW

The bridge that lets tools invoked inside the Senpi eval kernel bind to the parent session's kernel: capability validation, name normalization, nested invocation scope, and revived-child wrappers.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Barrel | `index.ts` |
| Bindings | `bindings.ts` - `createKernelToolBindings`, `KernelToolBindingRegistry` |
| Capability validation | `contract.ts` - structural checks over the live senpi kernel capability |
| Nested host scope | `nested-host-scope.ts` - `childInvokeScope`, `escalatingHostTools`, `isWriteCapableHostTool` |
| Names | `names.ts` (tool-name normalization), `transcript-names.ts` (transcript-facing names) |
| Resolution + wrappers | `resolve.ts`, `wrapper.ts` (unavailable-tool stubs for revived children) |

## CONVENTIONS

- The live senpi kernel capability is duck-typed because the pinned engine may predate it - validate structurally, never by importing the senpi codemode implementation.
- The optional invoke scope is sent only when the host advertises `capabilities.invokeScope === true`; gating is by the advertised marker, not a version guess.
- Error vocabulary is centralized; wrappers surface unavailable bindings as typed errors with the error codes defined here.

## ANTI-PATTERNS

- NEVER let a revived-child wrapper invoke a missing parent kernel binding - it answers the typed unavailable error instead.
- Do not assume invoke-scope support; always gate on the advertised capability.
- Do not import the senpi codemode implementation from here (keep the package importable without a live Senpi).

## COMMANDS

```bash
bun test --timeout 20000 packages/senpi-task/src/kernel-tools
bun run --cwd packages/senpi-task typecheck
```

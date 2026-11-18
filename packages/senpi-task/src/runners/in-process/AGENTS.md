# src/runners/in-process/ - InProcessRunner Internals

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 24 (34 files). The facade is sibling `../in-process.ts` (`InProcessRunner`); execution-mode semantics and the shared-tool invariants live in [`../../../runners/AGENTS.md`](../AGENTS.md) and the package parent - not repeated here.

## OVERVIEW

Child handles, session/options construction, kernel-tool scoping, and the curated read-only broker used by in-process children.

## WHERE TO LOOK

| File | Role |
|------|------|
| `child-handle.ts` | Child session/handle contracts + create/restore factories - a restored child is rebuilt from its persisted transcript; the original prompt is NEVER replayed |
| `child-options.ts` | `buildChildSessionOptions`, `requireChildSessionDir`, input/tool-name helpers |
| `child-loader.ts` / `create-child-session.ts` | Child loading and session creation |
| `shared-tool-filter.ts` | `filterSharedParentTools` - the parent-closure filter (own suite) |
| `host-tools.ts` / `kernel-tool-surface.ts` | Kernel tool scope + surface (python/revive suites) |
| `curated-readonly-bash.ts` | The shell-free gh/curl broker that replaces the shell for curated read-only children |
| `native-run-pause.ts` | Native pause wiring for in-process children |
| `subagent-prompt.ts` | Subagent prompt construction |
| `session-file-guard.ts` / `pin-check.ts` | Session file guarding, pin checks |
| `runner-error.ts` / `runtime-fallback-settings.ts` | Typed runner errors; fallback settings |
| `__fixtures__/` | Local fixtures; RPC-host fixtures live in `../rpc-host/__fixtures__/` |

## ANTI-PATTERNS

- A restored child never replays the original prompt - rebuild comes from the persisted transcript alone.
- Tool parity between parent and child is pinned by `builtin-tool-parity.integration.test.ts` and `child-tool-surface.test.ts`; do not widen the child surface without those suites.
- Curated read-only children get the broker, not the shell - don't hand execution back.

## COMMANDS

```bash
bun test packages/senpi-task/src/runners/in-process
bun run --cwd packages/senpi-task typecheck
```

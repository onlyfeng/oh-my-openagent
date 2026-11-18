# senpi-desktop-protocol - Engine Wire Contract

**Generated:** 2026-11-18 / c2691818a9

**Score:** 16 (17 files, ~1.0k LOC). The typed JSON-RPC contract for the desktop engine: method/param types, session snapshot, error codes, and the computer-call approval tiers. Imports NO workspace package - it is the leaf of the desktop family. Parent: [`packages/AGENTS.md`](../AGENTS.md) (desktop-family row).

## STRUCTURE

| File | Owns |
|------|------|
| `src/engine-schema.generated.ts` | GENERATED from the engine method table by `scripts/generate-engine-schema.ts` (`bun run generate`) - never edit by hand |
| `src/wire.ts` / `src/json-rpc.ts` / `src/port.ts` | wire types, JSON-RPC framing, `RpcPort` + `RpcPortFailure` / `RpcCallOptions` / notifications |
| `src/call.ts` / `src/session.ts` | call path, session snapshot |
| `src/audit.ts` | `AuditRecord`, `AuditStatus`, primary error |
| `src/index.ts` | barrel incl. `DESKTOP_METHODS`, `WINDOW_METHODS`, `DISPLAY_METHODS`, `ELEMENT_METHODS` |

## TESTS (`test/`, vitest - own `vitest.config.ts`)

- `schema-drift.test.ts` - the generated schema matches the engine method table.
- `method-allowlist.test.ts` - every model-facing computer-call method is allowlisted against the generated `CHAIN_METHOD_SNAPSHOT`; the snapshot is generated from the prelude's facade manifest (`bun run generate`; edit CHAIN_BINDINGS there, not here).
- `call.test.ts` - call-path behavior.

## CONVENTIONS

- Import direction is law (README "Import direction"): this package imports no workspace package; `-engine` may import this; only omo-senpi's computer-use component consumes the family (via `-tool`/`-service`); NO desktop package imports `pi-agent-core`/`pi-ai`/`pi-tui`.
- Schema changes start in the engine method table, then `bun run generate` - never in the generated file.

## ANTI-PATTERNS

- Never hand-edit `engine-schema.generated.ts`.
- Never add a workspace dependency; this package stays the leaf.

## COMMANDS

```bash
bun run test:desktop        # includes this package's vitest suites
bun run generate            # regenerate the schema (from the package dir)
```

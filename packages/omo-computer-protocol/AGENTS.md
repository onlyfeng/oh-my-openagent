# omo-computer-protocol - Computer-Use v3 Wire Types (Generated)

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 23 (12 files; generated surface consumed by the senpi-desktop/omo-computer TS families). The parent role map has the one-row placement.

## OVERVIEW

Generated wire types, call-chain classifier, and schema-drift test for the computer-use v3 engine: TypeScript mirror of `crates/omo-computer-core/schema/engine.schema.json`. Private, ESM, exports TS source.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `src/index.ts` | Barrel: `packageName`, `dependencyPackageNames`, `CallStep`, `ChainVerdict`, `classifyCallChain`, `resolveOneshot` |
| `src/engine-schema.generated.ts` (~1.8k lines) | `SCHEMA_SHA256`, `PROTOCOL_VERSION`, `ABI`, `METHODS`, `NOTIFICATIONS`, `ONESHOT`, `ENV_SEAMS`, `SETTINGS_TABLE`, `MethodParams`/`MethodResults`, wire types |
| `src/call-chain.ts` | Read/exec tier classification + one-shot aliases |
| `scripts/generate-protocol.ts` | The generator (run via `bun run generate`) |
| `test/` | Schema-drift suite (vitest) - fails when the generated file is stale |

## CONVENTIONS

- `SCHEMA_SHA256` supports drift detection: regenerate after ANY schema change, and the drift test stays green.
- The classifier distinguishes read vs exec tiers and one-shot aliases - tier decisions come from it, never ad hoc.

## ANTI-PATTERNS

- NEVER hand-edit `engine-schema.generated.ts` - regenerate (`bun run generate`) after changing the Rust schema.
- Never widen the export map; consumers get the barrel.

## COMMANDS

```bash
bun run --cwd packages/omo-computer-protocol typecheck   # tsgo --noEmit
bun run --cwd packages/omo-computer-protocol test        # vitest --run test
bun run --cwd packages/omo-computer-protocol generate    # regenerate from the Rust schema
```

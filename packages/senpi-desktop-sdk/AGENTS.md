# senpi-desktop-sdk — Typed Computer-Use SDK

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 29 (37 files; 20 modules under `src/` incl. `transport/`). One-row placement in the parent role map; nothing repeated here.

## OVERVIEW

Typed SDK over an engine `RpcPort`: `connect()`, the `Computer`/`Window`/`Element` facade, structured `ComputerError`, and the generated facade manifest. Package `@oh-my-opencode/senpi-desktop-sdk` (private, ESM, TS-source exports, depends on `senpi-desktop-protocol`).

## WHERE TO LOOK

| File | Role |
|------|------|
| `src/index.ts` | Barrel: `connect`, `Transport`/`ConnectOptions`, `ComputerError`, `ConfirmControl`, facade interfaces, action/report/selector/geometry types, policy constants (`BACKGROUND`/`FOREGROUND`/`SEMANTIC`) |
| `src/connect.ts` | `connect(transport)` — three transport kinds: `inProcess` wraps a port the host already owns (`close()` detaches, port stays with its owner); `spawn` starts an engine child via injected `launch` (SDK owns the port, `close()` ends it); `attach` joins a running `--serve` daemon socket. `toolBridge` is not implemented |
| `src/facade.ts` | The contract: every interface method carries TSDoc `@tier`, `@policy`, `@wire` — policy is encoded in tags, never inferred from names |
| `src/computer.ts` / `window.ts` / `element.ts` / `frame.ts` / `control.ts` / `batch.ts` | The facade surface (pointer, keyboard, menus, frames, confirm control, batching) |
| `src/transport/spawn.ts` / `attach-uds.ts` | Spawn launcher + UDS attach transports |
| `src/errors.ts` | Structured `ComputerError` |
| `src/validate.ts` / `verify.ts` / `audit.ts` / `epoch.ts` / `report.ts` / `select.ts` / `target.ts` / `link.ts` / `types.ts` | Boundary validation, verification, audit parsing, epochs, reports, selectors, targets |
| `scripts/generate-facade-manifest.ts` → `facade-manifest.json` | Generated manifest of tagged facade methods (checked in) |

## CONVENTIONS

- Boundary replies are validated; input operations return `ActionReport`, errors are structured `ComputerError` values.
- Importing the package has no side effects.

## ANTI-PATTERNS

- Every facade method signature must carry `@tier`, `@policy`, `@wire` — unsupported contract edits are rejected by manifest generation, never silently omitted.
- Facade policy lives in the TSDoc tags + manifest — do not duplicate it as hand-written runtime tables.

## COMMANDS

```bash
bun run --cwd packages/senpi-desktop-sdk typecheck    # tsgo --noEmit
bun run --cwd packages/senpi-desktop-sdk test         # vitest --run test
bun run --cwd packages/senpi-desktop-sdk generate     # regenerate facade-manifest.json
```

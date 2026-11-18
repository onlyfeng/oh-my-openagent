# src/runners/rpc/ - RPC Child-Process Runner

**Score ~16** (66 files incl. tests/fixtures; child-process lifecycle + wire protocol domain).

## OVERVIEW

The RPC runner for senpi-task: spawns and steers a newline-delimited-JSON child process (the Senpi adapter runner), normalizes wire errors, maps child exits, records crashes, and adjudicates model admission. The process/protocol core lives in sibling modules (`rpc-process`, `protocol-client`); this directory holds delivery semantics, lifecycle handles, error/exit mapping, admission, and the test fixtures that drive them.

## WHERE TO LOOK

| File | Role |
|------|------|
| `errors.ts` | Wire error codes/provider ids (lowercase shape regexes), RPC error classes |
| `protocol-errors.ts` | Protocol-level error normalization |
| `delivery-semantics.ts` | `RpcStreamingBehavior`, `isBusyChildRejection` |
| `exit-mapping.ts` | Child-exit → outcome mapping |
| `crash-record.ts` | Crash-record input + writers |
| `handle.ts` | Run handle (steer, dispose, wake sources, exit order) - heavily tested |
| `terminate.ts` / `start-cleanup.ts` | Termination + cleanup |
| `spawn.ts` / `senpi-launcher.ts` | Child spawn + launcher |
| `model-admission.ts` / `model-catalog-probe.ts` | Model admission adjudication + catalog probe |
| `turn-settlement.ts` / `turn-claim.ts` / `turn-outcome.ts` | Turn lifecycle settlement |
| `heartbeat.ts` | Heartbeat |
| `__fixtures__/` | `fake-child.mjs`, `spawn-fake.ts`, `silent-child.ts`, fake clock, Windows console probes (incl. `.ps1` interop) |

## CONVENTIONS

- Fixtures model the newline-delimited JSON RPC child protocol; fake-child switches `FAKE_IGNORE_TERM=1` (exercise SIGKILL escalation) and `FAKE_EMIT_UI=1` (startup confirmation) are fixture controls, not production behavior.
- Windows console behavior is isolated in dedicated probe fixtures (PowerShell interop helper + Bun FFI probe).
- Bun runner, colocated `*.test.ts`, `#given/#when/#then` names.

## ANTI-PATTERNS

- Treat the wire error codes and provider-id shapes as a closed contract - extend via the regexes in `errors.ts`, not ad-hoc strings.
- Fixture toggles (`FAKE_*`) must never leak into production spawn paths.

## COMMANDS

- `bun test packages/senpi-task/src/runners/rpc` · `tsgo --noEmit -p packages/senpi-task/tsconfig.json`

# desktop-sync/ - Desktop Memory-Sync Channel

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 21 (26 files, ~2.7k LOC; nested under `components/memory/`). Memory-component anatomy lives in [`../AGENTS.md`](../AGENTS.md).

## OVERVIEW

The desktop memory-sync seam: signed-in account binding, typed wire protocol, worker port, launch classification, and the operation handles that scope sync requests.

## WHERE TO LOOK

| File | Role |
|------|------|
| `desktopSyncAccount.ts` | `openDesktopSyncAccount(wire, home?)` - account, authority, transport, and genesis bridge (uses memory-core types + the light `sync-capabilities` value entry) |
| `channel.ts` / `framing.ts` / `workerPort.ts` | The typed channel over the worker port; framing |
| `desktopSyncProtocol.ts` / `protocol.ts` | Strict Zod wire schemas - request/event shapes and types |
| `desktopSyncOperation.ts` | Operation handles scoping sync requests |
| `bootClaim.ts` / `bootGate.ts` / `desktopClaim.ts` | Boot/desktop authority claims and gates |
| `launchClassification.ts` / `hostSelection.ts` | Launch classification (desktop vs engine) and host selection (+ fixtures) |
| `desktopSyncPortSchema.ts` / `desktopSyncReplicationPort.ts` / `desktopSyncBoundary.ts` / `desktopSyncAccountPorts.ts` | Port schemas, replication port, boundary, account ports |

## ANTI-PATTERNS

- A value import from the memory-core sync engine pulls the whole engine + wasm into the bundle - use the light `sync-capabilities` subpath.
- Private channel authority stays in the desktop main process; local shape validation is not full trust - the main side revalidates prepared genesis before signing.
- Memory words stay inside the protected desktop window; only public prepared genesis returns.
- Not the CLI credential source - do not wire one in.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/memory/desktop-sync
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```

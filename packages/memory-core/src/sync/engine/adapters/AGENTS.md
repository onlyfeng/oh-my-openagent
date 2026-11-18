# adapters/ - Sync Engine Adapter Seam

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 18 (12 files; the port factories the sync engine consumes). The sync security boundary and subdir pointers live in [`../../AGENTS.md`](../../AGENTS.md).

## OVERVIEW

Implementations of the sync engine's `ItemStore` / `SyncRepo` / `Transport` ports at the filesystem, Git, and HTTP boundaries.

## WHERE TO LOOK

| File | Role |
|------|------|
| `device-item-store.ts` | `createDeviceItemStore` - CRDT item snapshots + materialized state under `<repo>/.git/omo-sync/crdt/`; writes via temp file + rename |
| `git-sync-repo.ts` | `createGitSyncRepo` - private index, compare-and-swap HEAD update, `SyncCommitError`; never requires a clean working tree |
| `http-transport.ts` | `createHttpTransport` - `SyncHttpError`/`SyncAuthError`/`SyncNetworkError`; digest conversion `b64urlOfHex`/`hexOfB64url` |
| `device-item-recovery.ts` | `createItemRecovery` - `Baseline`/`ItemStoreState` recovery |
| `git-sync-repo.test-support.ts`, `wire-server.test-support.ts`, `qa-device.test-support.ts`, `qa-repo.test-support.ts` | Test supports: repo fixtures, a wire server, QA device/repo stands |

## CONVENTIONS

- Digests are base64url on the wire, hex inside the engine - convert only at this seam (`http-transport.ts`).
- Item plaintext snapshots live in the repository's Git directory, not the worktree.

## ANTI-PATTERNS

- Git sync must NOT require a clean working tree - unrelated uncaptured edits stay dirty and unstaged.
- HEAD moves only via compare-and-swap - never a bare reset/checkout in the sync path.
- Adapter boundaries return typed errors; do not let raw fetch/git errors cross into the engine.

## COMMANDS

```bash
bun test packages/memory-core/src/sync/engine/adapters     # focused
bun run --cwd packages/memory-core typecheck               # tsgo --noEmit
```

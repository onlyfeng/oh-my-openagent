# src/cli/service-auth/keychain-helper/ - macOS Keychain Helper Protocol

## OVERVIEW

41 TypeScript files. The signed-helper protocol for macOS Keychain access in
service auth: pinned helper executable, spawn/verify lifecycle, locked store
binding, migration and recovery. Consumed by `../keystore.ts` (imports
`client`, `darwin-backend`, `darwin-store`, `store-binding` directly - there
is NO barrel; consumers import module paths). Distinct domain: OS secret
storage. Parent: [`src/cli/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| Area | Role |
|------|------|
| `client.ts`, `access.ts` | `callHelper`/`spawnDetachedHelper`, protocol types; `resolveHelperAccess` pinned-copy verification |
| `store-binding.ts` | `bindKeychainHelper`, `lockedStoreView`, `LockedStore` - store integration seam |
| `darwin-backend.ts` | Backend creation/access; keystore error mapping |
| `darwin-read.ts`, `darwin-store.ts` | Locked read + store, approval-before-lock, migration/recovery |
| `pin.ts` | Signed-copy pinning; verified before every spawn |
| `location.ts` | Helper discovery (fixed locations only) |
| `owner-marker.ts` | Attribute-only ownership probe |
| `*.test-support.ts` | Fake keychain / process-copy helpers |

## CONVENTIONS

- OS/process/signature boundaries use explicit injected callbacks/options;
  helper replies validated by a strict Zod schema.
- Mutating verbs (`write`, `stage`, `retire`, `promote`, `remove`) must hold
  the session-lock lease (`client.ts`).
- macOS paths joined with POSIX separators even on non-macOS hosts.

## ANTI-PATTERNS

- NEVER discover the helper through PATH or environment variables
  (`location.ts`).
- NEVER execute an unverified or changed helper copy - verify the pin before
  every spawn (`access.ts`).
- `owner-marker.ts` probes attributes only - never read the secret or open a
  dialog from a probe.
- Keep approval before the lock; locked reads are non-interactive.

## QA

```bash
bun test --timeout 20000 packages/omo-opencode/src/cli/service-auth/keychain-helper
bun run --cwd packages/omo-opencode typecheck
```

# service-auth - Hidden CLI Login + Credential Store (direct files)

**Score:** 16 (75 direct files - 27 production, 40+ tests/fixtures; curated barrel). Scope note: this file covers the directory's own files only; `keychain-helper/` (nested) is out of scope here. Parent: [`src/cli/AGENTS.md`](../AGENTS.md) (the hidden `service-auth login` command).

## OVERVIEW

Service authentication for the omo account: device/loopback/browser login flows, the credential keystore, sessions with key-discard hooks, and the protocol schemas they share. `index.ts` is the curated public barrel; implementation modules export more than the barrel exposes on purpose.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Public barrel | `index.ts` - `accountHome`, `credentialLockPaths`, `withCredentialLock`, `currentServiceDevice`, `createCredentialStore`, `createSession`, `SignInError`, `retireSyncPinOnDiscard` |
| CLI command wiring | `commands.ts` - `configureServiceAuthCommands`, `serviceAuthProgram`, `runServiceAuthCommand`, `beginLogin` |
| Login flows | `device-flow.ts`, `loopback.ts` (`loginLoopback`), `browser.ts` (`openSignInBrowser`), `closed-device-login.ts`, `machine-device.ts` |
| Credential store | `keystore.ts` (`CredentialStore`, `KeychainApprovalPendingError`, `quietNotice`), `credential-lock.ts`, `credential-owner.ts`, `secret-service-lock.ts` |
| Sessions + discard | `session.ts` (`Session`, `BeforeKeysDiscarded`, `DiscardedCredential`), `owned-refresh.ts`, `refresh-attempt.ts`, `sync-pin-retire.ts` |
| Protocol | `protocol.ts`, `protocol-v2.ts`, `protocol-request.ts`, `machine-digest.ts`, `device-keys.ts`, `device-name.ts` |
| Gates + state | `accounts-gate.ts`, `account-home.ts`, `cli-service-auth-state.ts`, `signed-out-marker.ts`, `recovery-clock.ts`, `terminal-text.ts` |

## CONVENTIONS

- Focused kebab-case modules; colocated `*.test.ts`, `*.fixture.ts`, and `test-support.ts`.
- Tests inject fake APIs, temporary homes, and fake clocks; integration tests call `buildServiceAuth(false, ...)` from `script/build-service-auth` in `beforeAll`.

## ANTI-PATTERNS

- Tests must never touch the host credential store (closed-accounts fixtures enforce the gate).
- Test logins must never install or link a real device service.
- Do not widen the barrel to re-export implementation internals; consumers go through `index.ts`.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-opencode/src/cli/service-auth
bun run --cwd packages/omo-opencode typecheck
```

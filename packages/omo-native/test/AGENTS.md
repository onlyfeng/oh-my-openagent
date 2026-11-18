# test/ - omo-native package-contract + launcher tests

**Generated:** 2026-11-18 / c2691818a9

95+ bun:test suites (~38k LOC) over the `bin/` launcher, remote/account/daemon/compiled-launch, trust, keychain and setup surfaces. Score 14, distinct domain: this is the package's entire verification surface with its own enforced conventions, while the parent [packages/omo-native/AGENTS.md](../AGENTS.md) `test/` row only names `pty-signal-qa.py`.

## WHERE TO LOOK

| Need | File |
|------|------|
| Temp-root cleanup | `teardown.test-support.ts` - `teardownRoots(roots)` (the `afterEach` norm), `rmSyncEbusyTolerant`, `withDatabase`, `closeTrackedDatabases`, `TEARDOWN_FAILURE_PREFIX` |
| Spawn-site enumeration | `spawn-call-sites.ts` - `spawnCallSites(repoRoot, files)` (AST + binding resolution) |
| Bun binary fixtures | `bun-bin-shim.fixture.ts` / `bun-bin-shim.test-support.ts` |
| Keychain harness | `keychain-helper.test-support.ts` |

## CONVENTIONS (enforced by the suites)

- Isolate everything: `mkdtempSync` homes / config roots / agent dirs registered for `teardownRoots`; shared real homes, global installs and the user's credential/session stores are never fixtures.
- `rmSyncEbusyTolerant` retries Windows `EBUSY` only, with escalating delays (50→400ms, 14 attempts, ~4.4s worst case); every other errno throws on the first attempt, and on POSIX it is byte-identical to `rmSync`. Close database handles first (`withDatabase`).
- Process-spawn call sites are found by parsing (oxc-parser) and resolving bindings - aliases, wrappers, re-exports, parameter propagation - never by matching call text.
- Secret values never travel over RPC or land in persisted output; credential-shaped env is scrubbed and restored around suites (see `category-coverage.test.ts`'s `CREDENTIAL_ENV` pattern).
- Real-engine suites (`daemon-real-engine`, `category-coverage`) run the pinned senpi with fixture HOME/agent-dir, use explicit test budgets, and track + kill every spawned child.
- Gates: POSIX-only OpenSSL/trust suites skip on win32; the `openssl` CLI is required when enabled; release-gated suites (e.g. `device-service-pin-served.test.ts`) need `OMO_RELEASE_GATE=1`.
- Engine record types derive from the pinned engine's method table at runtime (`remote-attach.test.ts`), never from a list kept in this package.
- Cross-package imports are intentional: `device-service-core`, `omo-senpi`, `omo-opencode` and `memory-core` conformance fixtures are exercised from here.

## COMMANDS

- Focused suite: `bun test packages/omo-native/test/<file>.test.ts` (whole-directory command in the parent AGENTS.md).
- Support modules (`*.test-support.ts`, `*.fixture.ts`, `spawn-call-sites.ts`) carry no `.test.` name - Bun does not pick them up as suites.

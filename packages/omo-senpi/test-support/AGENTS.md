# omo-senpi/test-support - Hostless Test Fakes

**Generated:** 2026-11-18 | c2691818a9

Shared test doubles for the Senpi adapter: a fake `ExtensionAPI` plus event helpers, subprocess and tempdir utilities. Imported by every component's `*.test.ts` so tests run without a live Senpi host. Not shipped - development-only.

## WHERE TO LOOK

| Path | Role |
|------|------|
| `fake-extension-api.ts` | `FakeExtensionAPI` + fake event helpers; records commands, flags, messages, tools, renderers, MCP registrations, dispatched results. `withInertSenpiSeams` neutralizes ambient side effects. |
| `spawn-timeout.ts` | `spawnTimeout` - bounded child-process helper for tests. |
| `split-local-session.ts` | Split-session fixture helpers. |
| `temp-dir.ts` | Temporary directory helper with cleanup. |

## CONVENTIONS

- Tests import from `omo-senpi/test-support`, never re-implement a fake.
- `FakeExtensionAPI` must record what a real `ExtensionAPI` would do - if a method returns void on the real API, the fake records the call and returns void; do not give it behavior the host does not have.
- `spawnTimeout` always bounds the child; never spawn a test child without a deadline.
- `temp-dir.ts` cleanup is explicit; tests are responsible for disposing their own fixture directories.

## ANTI-PATTERNS

- Not shipped. Nothing in this directory may be imported from production code.

## COMMANDS

```bash
bun test packages/omo-senpi/test-support
```

# local-api - Device-Service Local API Test Harness

**Score:** 18 (87 files, 9.1k LOC, >20 files, 34 protocol suites, pinned conformance fixtures). No package-level AGENTS.md exists; this is the test heart of [`packages/device-service-core/`](../../).

## OVERVIEW

Protocol conformance + adversarial tests for the device-service local API: typed fake owners, a conformance runner, pinned service captures, and 22 numbered protocol cases (`la1`…`la22`) covering version refusal, squatting servers, proofs, tokens, DACLs, chunking, and listing pagination.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Conformance runner + fake owner | `conformance/` - `runner.ts`, `fake-owner.ts`, `fake-owner-sessions.ts`, `fake-owner-state.ts`, `world.ts`, `fake-clock.ts`, `raw-peer.ts`, `rejection.ts`, `patterns.ts` |
| Numbered protocol cases | `la1-version-refusal.test.ts` … `la22-listing-row-durable-id.test.ts` (numbered order = protocol story order) |
| Security cases | `la3-squatting-server`, `la4-wrong-client-proof`, `la5-stale-token`, `la6-insecure-state-dir`, `la17-exact-windows-dacl` + `dacl-verdict.test.ts` |
| Pinned captures | `fixtures/` - `device-service-0.1.2x-omo-5.1.66-*.json` real service endpoint/handshake snapshots |
| Shared harness | `support.ts` (273 LOC), `event-examples.ts`, `conformance/fake-owner-handlers.ts`, `conformance/fake-owner-deliveries.ts` |

## CONVENTIONS

- Case files use the `laN-<concern>.test.ts` naming; the number pins the case identity across renames.
- Fake owners are typed TypeScript harnesses over JSON fixtures - no live service needed; `raw-peer.ts` speaks the wire for adversarial frames.
- Platform differences are explicit: Windows DACL checks (`la17`), Unix socket path limits in the transport tests.
- Largest suites: `la21-reply-unknown-fields.test.ts` (337 LOC) - unknown-field tolerance is a contract, not an accident.

## ANTI-PATTERNS

- Do not regenerate pinned `fixtures/` captures casually - they freeze real service versions (filenames carry them).
- Do not loosen `conformance/runner.ts` assertions to absorb a protocol change; add a new `laN` case instead.

## COMMANDS

```bash
bun test --timeout 20000 packages/device-service-core/test/local-api
bun run --cwd packages/device-service-core typecheck
```

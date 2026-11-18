# onboarding - First-Run Onboarding + Setup Scan (direct files)

**Score:** 16 (43 direct files, 8.2k LOC). Claim semantics (`claimOnboarding`, `--onboard`, #9917), the scan RPC stream design, and the in-memory digest rules are documented in the parent [`packages/omo-senpi/AGENTS.md`](../../../../AGENTS.md); `session-history/` has its own AGENTS.md and is out of this file's scope.

## OVERVIEW

The onboarding component (first-run flow) and the onboarding-scan component (`omo.onboarding.scan` / `apply` RPC). Direct files only: component wiring, claim machinery, state persistence, scan/understand contracts, the session-history bridge, and TUI helpers.

## WHERE TO LOOK

| Area | Files |
|------|-------|
| Entry + components | `index.ts` (`createOnboardingComponent`, `createOnboardingScanComponent`), `component.ts`, `scan-component.ts` |
| Claim machinery | `claim-owner.ts`, `claim-takeover-contender.ts`, `claim-race-child.mjs` (subprocess probe; pinned by `claim-takeover-race.test.ts`) |
| State | `state.ts` (markers, cooldown), `state-rpc.ts` (state RPC registration/contracts) |
| Scan surface | `scan-contract.ts`, `scan-runtime.ts`, `scan-session-sources.test.ts`, `tui-scan-runtime.ts` |
| Understand flow | `understand-component.test.ts` + understand contracts |
| Session-history bridge (top-level only) | `session-history-rpc.ts`, `session-history-tool.ts`, `session-history-tool-runtime.ts`, `session-history-runtime-loader.ts` |
| TUI memory tour | `tui-memory-flow`, `tui-memory-runtime.ts`, `tui-memory-tools.ts` |
| Test harness | `test-harness.ts`, `lifecycle/preview/release/migration-facts` suites |

## CONVENTIONS

- Scan loads lazily, once, only after a claim succeeds - never in `register()`; sessions that never onboard pay no scan cost (fail-open).
- `claim-race-child.mjs` is a real subprocess probe, not a mock: race tests spawn it to prove single-claim across processes.
- RPC/tool registrations are separate files per surface (state vs scan vs session-history); keep new surfaces in their own registration module.

## ANTI-PATTERNS

- Do not persist anything a scan captured before explicit consent - digests stay in memory with the retained scan.
- Do not add an unbounded read or parse a transcript format ad hoc in the session-history bridge (see `session-history/AGENTS.md`).

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-senpi/src/components/onboarding
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```

# test-support - shared test infrastructure

**Why this file exists:** score 13 (file count 3x, code ratio 2x, own knip config 1x, exports 2x, centrality 3x - >400 imports from 200+ test files across every package). Distinct domain: process-global guarantees wired into each package's `bunfig.toml`.

## OVERVIEW

Flat directory of bun:test helpers: telemetry blocking, environment integrity, Windows-safe tree removal, held-root leak reporting, test time budgeting, GitHub Actions expression evaluation. No subdirectories; colocated `.test.` files cover each helper.

## WHERE TO LOOK

| Need | File | Notes |
|------|------|-------|
| Block product telemetry from tests | `posthog-test-guard.ts` (+ `.preload.ts`) | Prototype-level fetch/capture/shutdown interception (#10394); loopback opt-in; `takePostHogTestGuardViolations()` for deliberate-refusal tests |
| Prove a test leaked env keys | `environment-integrity.ts` | `Bun.plugin` afterAll baseline per test file; `restoreEnvironment(snapshot)`; detects a replaced `process.env` object (#10846) |
| Snapshot/restore telemetry env vars | `telemetry-env.test-helpers.ts` | `isolateTelemetryEnv()` over 8 `OMO_*`/`POSTHOG_*`/`DO_NOT_TRACK` keys |
| Delete a tree on Windows | `remove-tree.ts` | `removeTree`/`removeTreeSync`, injectable `remove` op for failure simulation |
| Report (not crash on) a held temp root | `held-test-root.mjs` | `skippedTestRoots` exact-path map; `removeTestRootBestEffort` defers, shared with abandoned-root sweep (#11190) |
| Diagnose who holds a root | `windows-held-root.mjs` + `.ps1` | PowerShell holder query, win32 only, 30s timeout, JSON `{holders, diagnosticErrors}` |
| Wait for async test events | `test-deadline.ts` | `awaitWithinTestBudget`, `testBudgetSignal`, `useTestTimeout`; budget = 0.9x test timeout |
| Assert workflow expression outcomes | `github-expression.ts` | `evaluate`/`interpolate` for `script/ci-*` workflow-structure tests |
| Test-only unsafe cast | `unsafe-test-value.ts` | 200+ consumers |
| Knip entries/ignores for this dir | `knip.workspace.jsonc` | Composed by root `script/knip-config.ts` |

## CONVENTIONS

- Wiring: every package `bunfig.toml` preloads `../../test-support/posthog-test-guard.preload.ts` + `test-deadline.preload.ts` (`skills-loader-core` also adds `hermetic-home.preload.ts`); the monorepo-root runner installs guard + env check + deadline reset via root `test-setup.ts`. `omo-senpi`/`senpi-task` instead preload their own package-local `test-support/warm-lazy-runtime.ts` - a different directory, do not confuse them.
- ESM helpers ship `.mjs` + matching `.d.mts` declaration (`held-test-root`); keep both in sync.
- Cleanup APIs expose async and sync variants with injectable `remove`/`diagnose` options so tests simulate failures deterministically.
- Preloads are tiny dedicated modules (`posthog-test-guard.preload.ts` also sets `OMO_DISABLE_POSTHOG=true`); each has an installed/audit test.

## ANTI-PATTERNS (THIS PROJECT)

- NEVER assign a plain object to `process.env` - it detaches writes from the real environment `os.tmpdir()`/children read; set and restore keys (#10846).
- NO fixed 2-10s sleeps racing events - they false-fire on stalled runners; use the test's own budget via `awaitWithinTestBudget`/`testBudgetSignal` (#10814).
- DO NOT let a PostHog client outlive its test or send non-loopback traffic; tests proving transport opt into a loopback sink (`127.0.0.1`, `localhost`, `::1`) and shut clients down.
- DO NOT trust Bun `fs.rm` `maxRetries`/`retryDelay` (parsed but never retried, oven-sh/bun#41480) or its false success when a child vanishes (#11806) - always go through `remove-tree.ts` (retries `EBUSY`/`EPERM`/`EMFILE`/`ENFILE`/`ENOTEMPTY`, asserts removal).
- NO prefix-wide cleanup exemptions: `skippedTestRoots` records exact paths only; preload and tests must resolve the same `held-test-root.mjs` module instance or the leak check fails.
- Test-only casts go through `unsafeTestValue` - never widen production types for fixtures.
- `github-expression.ts` `==`/`!=` are case-sensitive, unlike real GitHub - tests must not rely on case-folding.

## COMMANDS

```bash
bun test test-support/          # all helper tests (~9 files)
bun test test-support/posthog-test-guard.test.ts   # focused
```

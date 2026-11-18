# src/__tests__/ - Plugin Perf Benchmarks

**Generated:** 2026-11-18 (updated)

## OVERVIEW

Plugin-level tests that intentionally break the co-located `*.test.ts` convention: they boot the whole plugin via `createPluginModule()` and measure init cost. The directory holds only this file and `perf/` - all other plugin tests live colocated next to their modules.

| Path | Measures |
|------|----------|
| `perf/plugin-init.test.ts` | Cold `createPluginModule()` init time |
| `perf/plugin-init-team-mode-resume-defer.test.ts` | Init cost when team-mode resume is deferred |

## WHY SEPARATE

These tests pay the full plugin-assembly cost per run, so they are kept out of the colocated suites that CI runs per-module; colocating them next to a module would both misattribute ownership and slow that module's focused test loop.

## FIXTURES (DO NOT EXPAND)

`perf/fixtures/in-tree/` is a synthetic project tree (20 dummy `*.ts` files under `src/app`, `src/lib`, `packages/pkg-one/src`) that benchmarks AGENTS.md walk-up discovery.

Its three `AGENTS.md` files (`in-tree/`, `in-tree/src/`, `in-tree/packages/pkg-one/`) are deliberate 1-line stubs (`# fixture root`, etc.). They are TEST DATA, not documentation. `/init-deep` and any doc tooling MUST leave them as single-line stubs; expanding them corrupts the benchmark baseline.

## COMMANDS

```bash
bun test packages/omo-opencode/src/__tests__/perf
```

## NOTES

- Perf regressions land as measured numbers in the PR body (before/after init ms), never as silent threshold bumps.
- The init benchmark is the early-warning gate for the staged initialization order documented in [`../AGENTS.md`](../AGENTS.md) `INITIALIZATION (createPluginModule())`.

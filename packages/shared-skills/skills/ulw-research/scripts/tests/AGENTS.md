# tests/ - ulw-research Script Test Support + Fixtures

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 13 (40 files, 7 subdirs; the support seam + fixtures for the `../` suites). Cluster map and invariants of the runtime itself live in [`../AGENTS.md`](../AGENTS.md).

## OVERVIEW

Shared test-support modules (imported directly by the `../` `*.test.ts` suites) plus the fixture worlds they run against.

## WHERE TO LOOK

| File | Role |
|------|------|
| `derive-clock.ts` | `derive` wrapper - ALL successful test derivations use caller-owned FIXED observation time (default `2026-10-23T08:00:00Z`); spec may override `now` |
| `contracts-support.ts` | Contracts test support |
| `delivery-barrier-session.ts` | Delivery-barrier session support |
| `fact-derive-support.ts` / `fact-unit-dimensions-support.ts` / `fact-unit-rules-support.ts` | Fact-derive/unit supports (import the `../` `.mjs` production modules directly) |
| `fidelity-values-support.ts` / `report-tools-support.ts` / `value-mode-support.ts` | Fidelity, report-tools, value-mode supports |
| `numeral-classes.ts` | Numeral classification support |
| `fixtures/` | `good-report.html`, `bad-report.html`, `reference-report.html`, `conversion-probe.html`, `sources-shapes.html`, `boxes.json`, `design-spec.md`, `fidelity-brief-{locked,pending}.md`, `fidelity/`, `assets/` |

## CONVENTIONS

- Supports import production `.mjs` modules (`fact-derive.mjs`, `fact-ledger.mjs`, `unit-rules.mjs`, `units.mjs`) directly - no build step, no re-exports.
- Runner is `bun:test`; same command as the parent.

## ANTI-PATTERNS

- Never derive with wall-clock `now` - a successful derivation must be reproducible; pass a fixed time.
- Do not copy fixture content into suites; reference the fixture files (they are the single source).

## COMMANDS

```bash
bun test --timeout 20000 packages/shared-skills/skills/ulw-research/scripts   # from repo root (covers tests/)
```

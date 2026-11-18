# src/__adversarial__/ - Seeded Chaos Simulation

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 15 (14 files, ~2.2k LOC; distinct domain - the chaos bench). The parent QA section owns what the bench asserts; this file maps the harness.

## OVERVIEW

Deterministic seeded simulation over the REAL engine composition (lifecycle/manager/store/completion/steering/runners): random actions and lifecycle mutations against fake process/host/runner/notifier implementations, judged by invariant collectors.

## WHERE TO LOOK

| File | Role |
|------|------|
| `chaos-harness.ts` (~300 LOC) | Wires the simulation: real components + fakes + temp store |
| `chaos-engine.ts` / `chaos-engine-factory.ts` / `chaos-drive.ts` | Engine loop, factory, iteration driver (`runIteration`, probes) |
| `chaos-actions.ts` (~290 LOC) | Random action execution; `chaos-lifecycle-actions.ts` - lifecycle mutations |
| `chaos-host-harness.ts` / `chaos-host-actions.ts` | Host-level chaos (`ChaosProcessTable`, `ChaosRunner`, `ChaosRpcRespawnRunner`) |
| `chaos-invariants.ts` (~280 LOC) | The oracle: `InvariantId` is `1..13`, `collectInvariantViolations` / `collectHostChaosViolations`, notification-epoch tracker, instrumented `ChaosNotifier` |
| `observing-store.ts` | `createObservingStore`, `isTerminalStatus`, store observations |
| `prng.ts` | `hashSeed`, `deriveSeed`, `RandomSource`, `WeightedChoice` - deterministic randomness |

## CONVENTIONS

- All randomness is seeded; `SEED=<label> bun test src/__adversarial__` reproduces a failing run exactly.
- Three suites: `chaos-bench.test.ts` (200 iterations), `chaos-host.test.ts`, `chaos-lifecycle.test.ts`.

## ANTI-PATTERNS

- Never introduce unseeded randomness or wall-clock time into the harness - reproducibility is the point.
- New invariants extend `InvariantId` and the collectors; never assert engine behavior ad hoc inside actions.

## COMMANDS

```bash
SEED=<label> bun test src/__adversarial__                       # from packages/senpi-task
SEED=<label> bun test src/__adversarial__/chaos-bench.test.ts   # focused bench
```

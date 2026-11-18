# scripts/qa/cu-eval/ - Computer-Use Eval Harness

**Score ~9** (9 files; distinct domain - computer-use evaluation/benchmark driver).

## OVERVIEW

A QA/eval harness for the computer-use (desktop) surface: defines eval tasks, drives them through an engine port across model "arms", and scores results against a stored baseline. `.mjs` ESM scripts run under Node/Bun; `cu-eval.test.mjs` is the hermetic self-check.

## WHERE TO LOOK

| File | Role |
|------|------|
| `run.mjs` | Eval run entrypoint |
| `eval-core.mjs` | Core eval loop/scoring |
| `tasks.mjs` | Eval task definitions |
| `model-arms.mjs` | Model "arm" (variant) configuration |
| `engine-port.mjs` | Engine abstraction port |
| `bench.mjs` | Benchmark driver |
| `live-macos.mjs` | Live macOS driver |
| `baselines/linux-x64-fake.json` | Stored baseline fixture |
| `cu-eval.test.mjs` | Hermetic self-test |

## CONVENTIONS

- Engine access goes through `engine-port.mjs` so the same eval core runs against fake and live engines.
- Baselines are stored JSON fixtures; the fake baseline (`linux-x64-fake.json`) keeps the self-test hermetic.

## ANTI-PATTERNS

- Keep the hermetic self-test (`cu-eval.test.mjs`) independent of a live desktop/engine; live runs go through the explicit `live-*.mjs` drivers.

## COMMANDS

- Part of the omo-senpi QA surface; see `packages/omo-senpi/scripts/qa/AGENTS.md` and the `senpi-qa` skill for live driving + evidence.

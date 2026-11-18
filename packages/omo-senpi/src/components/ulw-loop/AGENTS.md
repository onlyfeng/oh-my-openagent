# components/ulw-loop/ — Senpi ULW Loop Component

## OVERVIEW

25 TypeScript files, ~3,160 LOC. The Senpi-side ULW loop: session-scoped run
status (footer), run completion tracking, and continuation routing through
the idle coordinator; reuses the shared continuation-eligibility helpers
from `ulw-execute-continuation`. Entry `index.ts` exports
`createUlwLoopComponent` + `UlwLoopComponentOptions`. **Score 16**.
Parent: [`src/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | `register(pi, ctx)`; session-scoped status reads; continuation routing via the idle coordinator when available (hotspot, ~350 LOC) |
| `footer-status.ts`, `status-source.ts` | Footer status rendering and status sourcing |
| `run-completion.ts` | Run completion tracking |
| `session-scope.ts` | Session scoping of runs/status |
| `sdk/` | SDK helpers (root provisioning hint, context types) |
| `ulw-loop.test-support.ts` | Shared test harness |

## CONVENTIONS

- Fail closed when session identity is unavailable: never auto-continue a
  run whose ownership cannot be established.
- Unknown/failed/malformed status reads must NOT be treated as proof that
  no plan exists.
- Continuation budget is spent only after `agent_settled` — never at
  `agent_end` (abort and host retry/compaction state may still change).
- Steering goes through the SDK in a JS eval cell; no session ID or plan
  path is passed; never spawn the removed toolkit CLI or call the removed
  `omo_agent_toolkit` tool.

## ANTI-PATTERNS

- Do not treat a status-read error as "no plan".
- Do not decide continuation at `agent_end`.

## QA

```bash
bun test packages/omo-senpi/src/components/ulw-loop
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```

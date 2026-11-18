# prompt-async-gate/ - Internal Prompt Dispatch Internals

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 18 (13 files). The mandate - every internal `session.prompt` dispatch routes through `dispatchInternalPrompt()` - and the consumer-facing facade (`../prompt-async-gate.ts`) are owned by [`../AGENTS.md`](../AGENTS.md); this file maps the gate's internals.

## OVERVIEW

The machinery under the facade: per-session reservations, draining queues, pending-turn blocking, route resolution, and semantic dedupe.

## WHERE TO LOOK

| File | Role |
|------|------|
| `reservations.ts` | Per-session reservations (`getActiveReservation`, `setExpiredReservationHandler`) |
| `queue.ts` | The dispatch queue - module-level per-session prompt queues, draining/in-flight/timer maps; `package.json` `sideEffects` pins this module, tree-shaking must keep it |
| `session-idle-dispatch.ts` | `dispatchAfterSessionIdle` - the idle-dispatch critical path |
| `pending-tool-turn.ts` (+ `pending-tool-turn-metadata.test.ts`) | Whether in-flight assistant tool work blocks an injection |
| `prompt-message-state.ts` | Message-state predicates (timing-safe status/message inspection) |
| `route-resolver.ts` | Route resolution through injectable callbacks - harness-neutral |
| `semantic-dedupe.ts` | SHA-256 semantic dedupe of queued prompts |
| `recent-dispatches.ts` | Recent-dispatch accounting |
| `timing.ts` | Timing constants/helpers |
| `message-inspection-error.ts` / `types.ts` | Guarded inspection errors; shared types |

## CONVENTIONS

- Harness-neutral: unknown host message shapes are guarded (probe, don't throw); routes are injected, never hardcoded to a harness.
- Timers are `declare`d (`setTimeout`/`clearTimeout`) so the module runs under any runtime.

## ANTI-PATTERNS

- Reservation supersession must NOT match a user prompt; transient retry-owner matching stays narrow.
- Do not add a second dispatch path around the queue - duplicate injections are exactly what the gate exists to prevent.
- Bypassing `dispatchInternalPrompt()` anywhere in the plugin is a defect (parent invariant).

## COMMANDS

```bash
bun test packages/utils/src/prompt-async-gate
bun run --cwd packages/utils typecheck
```

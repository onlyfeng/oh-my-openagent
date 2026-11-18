# ulw-execute-continuation - Boulder-Work Continuation Component

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 11 (8 files, ~1.7k LOC, distinct domain: post-turn continuation of active ULW work)

## OVERVIEW

Keeps an active ulw-execute work plan running across turns: appends a steering reminder to user input when continuable boulder work exists, and after a settled agent turn re-derives a continuation directive from the boulder ledger. The OpenCode-side hook equivalent lives in `omo-opencode/src/hooks/ulw-execute/`; this is the Senpi component.

## FLOW

1. `input` (user-sourced only): resets continuation state, records the run-completion baseline (`runCompletion.observe(cwd, sessionId)`), and - when `findContinuableBoulderWork(cwd, sessionId)` finds work - transforms the text by appending `ULW_EXECUTE_STEERING_REMINDER` (read `.omo/boulder.json` + the active plan; finish every checkbox before unrelated work).
2. `agent_end`: RECORD ONLY - stashes the payload as `pendingRun`. The decision deliberately waits for `agent_settled`, the host edge that guarantees no automatic retry, compaction, or queued continuation will run.
3. `agent_settled`: skip when (in order) the run's outcome is terminal (`readAgentEndOutcome` → `blockedBy`, stop reason/abort/retry), the continuation cap is hit (`CONTINUATION_LIMIT = 8`), context is missing, no continuable work remains, or the work signature (`work_id:updated_at:completed/total`) is stale. Otherwise deliver the directive.

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Component factory, event wiring, skip logic, signature dedup |
| `boulder-eligibility.ts` | `findContinuableBoulderWork(cwd, sessionId)` - boulder + plan + checklist read |
| `agent-end-eligibility.ts` | `readAgentEndOutcome(payload)` - terminal-outcome classification |
| `run-completion.ts` | `createRunCompletionTracker` → `pi.lifecycle.markRunCompleted({runId})` (warn-only on failure) |
| `hidden-delivery.test.ts` | Delivery stays hidden (`display: false`, custom `omo-senpi:ulw-execute-continuation` type) |

## CONVENTIONS

- Delivery prefers `ctx.idleCoordinator.enqueue` (key `omo-senpi-ulw-execute-continuation`, source `boulder-continuation`) with an explicit `scheduleFlush()`; a refused enqueue logs - the continuation is derived state, the next boulder edge re-derives it. Without a coordinator it falls back to `pi.sendMessage`.
- The directive names concrete paths: `.omo/boulder.json`, `.omo/ulw-execute/ledger.jsonl`, the plan file, and the `senpi:`-prefixed session id.

## ANTI-PATTERNS

- Don't decide on `agent_end` - a late user abort can still mutate that payload; only `agent_settled` is safe.
- Don't deliver when the signature is unchanged: same work state twice means the turn made no progress.

## COMMANDS

```bash
bun test packages/omo-senpi/src/components/ulw-execute-continuation
```

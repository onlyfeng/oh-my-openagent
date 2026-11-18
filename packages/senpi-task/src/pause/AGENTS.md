# src/pause/ — Pause/Resume Domain

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 18 (12 files, ~1.5k LOC; consumed by manager/lifecycle and the DAG scheduler).

## OVERVIEW

Admission barrier, durable pause registry, run controller, and the engine ports that pause/resume run through.

## WHERE TO LOOK

| File | Role |
|------|------|
| `ports.ts` | Engine boundaries: `ChildPausePort`, `GuardedChild`, `ChildCreationPort`, `RunRef`, `RunPauseOwner`, `RELEASE_RUN_FENCE`, hold/resume/inherit receipts |
| `admission-barrier.ts` | `AdmissionBarrier` — closes synchronously BEFORE registry I/O; admission is ticketed and rechecked at final child creation |
| `registry.ts` | `PauseRegistry` — durable JSON via temp file + rename, mode 0600 |
| `run-controller.ts` | `RunPauseController` — pause/resume orchestration + durable recovery |
| `guidance.ts` | `addGuidance` / `changeGuidance` / `dispatchGuidance`, `Guidance`/`GuidanceError` |
| `native-child-pause.ts` | `nativeChildPausePort` / `NativeRunPause` — calls into the child engine |
| `admission-identity.ts` | Admission identity resolution (own suite) |

## ANTI-PATTERNS

- `startPaused` must return an unstarted child — never one whose initial prompt already ran; no producer and no first prompt before the final admission check.
- Native pause calls go to the child engine, never a local stand-in.
- No I/O before the synchronous barrier close; reject unsafe session ids BEFORE constructing persistence paths.
- Domain state and engine ports stay separate from persistence/orchestration — do not fold `ports.ts` types into the registry.

## COMMANDS

```bash
bun test packages/senpi-task/src/pause
bun run --cwd packages/senpi-task typecheck
```

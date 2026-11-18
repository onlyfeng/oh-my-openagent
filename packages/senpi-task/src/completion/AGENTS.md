# src/completion/ — Completion Notification

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 20 (19 files; barrel consumed by the task manager and re-exported from the package barrel). Routing table and batched-delivery semantics are owned by [`../../AGENTS.md`](../../AGENTS.md).

## OVERVIEW

Exactly-once completion notifier, message rendering, and the routing predicates behind it.

## WHERE TO LOOK

| File | Role |
|------|------|
| `notifier.ts` | `createCompletionNotifier` — dedupe by `(task_id, run_epoch)`, retry ladder, `reconcileUnnotifiedNotifications` redelivery |
| `notification.ts` | `buildCompletionDetails`, `buildCompletionMessage`, `completionMessageLines`, `BuildDetailsOptions` |
| `routing.ts` | `routeCompletion`, `shouldNotifyStatus` — the routing predicates |
| `dag-verification-directive.ts` | DAG verification directives re-exported by the barrel |
| `recovery-failure-notice.ts` | Notice construction for recovery failures |
| `types.ts` | Shared notifier/message contracts |
| Tests | Suites are named after the invariant they pin: `notifier-retry`, `notifier-buffer-dedupe`, `notifier-reconcile-unnotified`, `notifier-delivery-failure`, `notifier-unrequested-interrupt`, `unconditional-wake`, `model-visibility`, `model-event-visibility`, `routing` |

## ANTI-PATTERNS

- Parent-initiated cancel/interrupt return synchronously and NEVER push a completion notification; only externally-caused `completed`/`error`/`lost` notify (`shouldNotifyStatus`).
- Idle parents always wake; streaming parents receive delivery — do not add a route that skips either.
- An unrequested-interrupt notification is a bug, not a feature — `notifier-unrequested-interrupt.test.ts` pins it.

## COMMANDS

```bash
bun test packages/senpi-task/src/completion
bun run --cwd packages/senpi-task typecheck
```

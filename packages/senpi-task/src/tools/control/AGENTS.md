# tools/control/ — `task_send` / `task_cancel` (senpi-task)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 13 (24 files, own barrel; sibling precedent [`../task/AGENTS.md`](../task/AGENTS.md)). Parent [`../../../AGENTS.md`](../../../AGENTS.md) pins the public tool names and factories; this file maps the steering surface.

## STRUCTURE

| File | Role |
|------|------|
| `send.ts` | `createTaskSendTool` / `runTaskSend` / `createMemberScopedTaskSendTool` — steering, resident-session revival, team routing (`TaskSendTeamRouting`), late-send outcomes (`LateSendOutcome` / `LateSendDelivery`) |
| `send-schema.ts` / `send-results.ts` / `send-shutdown.ts` | Param schema; result shapes; shutdown approval traffic (`send-shutdown.test.ts` pins the handshake) |
| `cancel.ts` | `createTaskCancelTool` / `runTaskCancel` |
| `renderers.ts` (+ `renderers.test.ts`, `renderers-result.test.ts`, `member-scoped-renderer.test.ts`) | Tool-result rendering for send/cancel |
| `tool-result.ts` | `toolResult` / `toolErrorResult` / `finalResponseHead` / `isTerminalStatus` — the one result envelope |
| `clamp.ts` | `clampWaitTimeout` (`WaitBounds`) — every wait knob is clamped here |
| `caller-session.ts` / `types.ts` | `defaultResolveCallerSessionId`; manager ports (`SendManager`, `CancelManager`), carrier types |

## CONTRACT

`task_send` is the only steering surface: it routes to a live or resident child, delivers queued (prelaunch) sends through the record's `pending_steering`, and carries team traffic to members. The member-scoped variant is the only send a member session receives (team-scoped `task_send`); the lead family never reaches members. `task_cancel` walks the same manager ports.

## CONVENTIONS

- Errors are data: `toolErrorResult` + typed details, never thrown out of `execute`.
- Terminal checks go through `isTerminalStatus`; wait values through `clampWaitTimeout` — no inline bounds.
- Renderer changes ship with `renderers*.test.ts` expectations (TUI text is pinned).

## ANTI-PATTERNS

- Do not add a second send path around the manager ports; `runTaskSend` is the one body.
- Do not widen the member-scoped tool beyond team-scoped send (the lead-only family stays lead-only).

## COMMANDS

```bash
tsgo --noEmit -p packages/senpi-task/tsconfig.json
bun test packages/senpi-task/src/tools/control
```

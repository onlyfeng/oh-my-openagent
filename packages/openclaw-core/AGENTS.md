# openclaw-core — OpenClaw Gateway + Reply Daemon (Core)

**Generated:** 2026-06-16 / updated 2026-10-13 (dc373d26f)

## OVERVIEW

Package: `@oh-my-opencode/openclaw-core`. Harness-neutral Core for OpenClaw bidirectional integration. Outbound dispatch fires HTTP webhooks or shell commands on session events; inbound reply-listener daemon polls Discord/Telegram and injects replies into tracked tmux panes. Consumed by the OpenCode adapter at [`packages/omo-opencode/src/openclaw/`](../omo-opencode/src/openclaw/AGENTS.md) via re-export shims.

## KEY FILES

| File | Role |
|------|------|
| `src/index.ts` | Barrel: `wakeOpenClaw()`, `initializeOpenClaw()`, `startReplyListener`, `stopReplyListener` |
| `src/types.ts` | `OpenClawConfig`, `OpenClawContext`, `OpenClawPayload`, `WakeResult` (imported by `omo-opencode/src/config/schema/openclaw.ts`) |
| `src/config.ts` / `src/gateway-url-validation.ts` | `resolveGateway`, `normalizeReplyListenerConfig`; `validateGatewayUrl` |
| `src/dispatcher.ts` | HTTP POST + shell command execution with variable interpolation |
| `src/runtime-dispatch.ts` | Maps harness events to OpenClaw events; orchestrates dispatch |
| `src/session-registry.ts` | JSONL registry correlating message IDs to sessions and panes; `-lock.ts` (`withRegistryLock`/`OrWait`), `-paths.ts` (storage dir, lock timeouts, `MAX_AGE_MS`), `-storage.ts`, `-types.ts` (`SessionMapping`) |
| `src/reply-listener.ts` | Daemon lifecycle barrel (start/stop/poll loop/status/log) |
| `src/reply-listener-start.ts` / `-spawn.ts` / `-startup.ts` | Detached Bun spawn, startup token + ready wait (`getReplyListenerStartupTimeoutMs`) |
| `src/reply-listener-stop.ts` / `-status.ts` / `-process.ts` / `-signature.ts` | Stop + exit wait, daemon identity marker check, runtime signature (restart on config/runtime change) |
| `src/reply-listener-state.ts` / `-paths.ts` / `-log.ts` | PID/state/config/log files under the state dir, written with `REPLY_LISTENER_SECURE_FILE_MODE` |
| `src/reply-listener-poll-loop.ts` | Polls every `pollIntervalMs` (default 3000, doubled after an error); prunes stale registry entries hourly |
| `src/reply-listener-discord.ts` / `-telegram.ts` | `pollDiscordReplies`, `pollTelegramReplies` |
| `src/reply-listener-injection.ts` | Injects received replies into tmux panes |
| `src/tmux.ts` / `src/tmux-path.ts` | `captureTmuxPane()`, `sendToPane()`, `analyzePaneContent()`; cached `getTmuxPath()` |
| `src/daemon.ts` | Entry point for the detached daemon process |

## FLOW

```
Discord/Telegram API → reply-listener-poll-loop.ts
  → session-registry.ts: lookup tmux pane by message ID
  → reply-listener-injection.ts: rate limit check → sanitize → send-keys into pane
```

## NOTES

- URL validation requires HTTPS except localhost (`gateway-url-validation.ts`).
- Session registry is a file-locked JSONL written via `session-registry-storage.ts`.
- Daemon state persists to disk via `reply-listener-state.ts` (PID, config signature, poll tracking).
- Rate limiter: one `ReplyListenerRateLimiter` per poll loop, `replyListener.rateLimitPerMinute` (default 10), shared by Discord and Telegram polling.
- `tmux.ts` pane confidence check skips injection if `analyzePaneContent()` returns below 0.3.
- Package exports `./*` map straight to `src/*.ts`; OpenCode shims import subpaths (`@oh-my-opencode/openclaw-core/<module>`). Deps: `tmux-core`, `utils` only.
- Tests: colocated `src/*.test.ts` plus lifecycle/restart suites in `src/__tests__/`; they isolate env and daemon files. `bun run --cwd packages/openclaw-core test` / `typecheck`.

Parent: [`packages/AGENTS.md`](../AGENTS.md)

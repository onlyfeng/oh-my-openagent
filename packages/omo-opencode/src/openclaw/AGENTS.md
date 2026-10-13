# src/openclaw/ — Bidirectional External Integration

**Generated:** 2026-05-15 | **Updated:** 2026-10-13 / dc373d26f

## OVERVIEW

This module provides a bidirectional integration system: **outbound** session event notifications (Discord/Telegram/HTTP webhook/shell command) AND **inbound** reply handling (daemon polls chat apps, injects replies back into tmux session). Harness-neutral gateway, daemon, session registry, and tmux injection primitives are extracted to [`packages/openclaw-core/`](../../../../packages/openclaw-core); this directory keeps OpenCode startup/event wiring stable.

## BIDIRECTIONAL FLOW

### Outbound (OpenCode → External)
```
OpenCode session event → dispatchOpenClawEvent()
  → runtime-dispatch.ts: map event to OpenClaw event
  → openclaw-core dispatcher.ts: execute gateway (HTTP POST or shell command)
  → openclaw-core session-registry.ts: record message ID ↔ sessionID ↔ tmux pane
```

### Inbound (External → OpenCode)
```
Discord/Telegram API → reply-listener daemon (separate Bun process)
  → reply-listener-{discord,telegram}.ts: poll every 3s
  → openclaw-core session-registry.ts: look up target tmux session from message ID
  → openclaw-core reply-listener-injection.ts: send-keys into tmux pane (rate limited)
```

## KEY FILES

Every module `.ts` file here (22, excluding the 19-line `__tests__/config.test.ts`) is a one-line `export * from "@oh-my-opencode/openclaw-core/<same-name>"` shim kept so adapter import paths stay stable. ALL logic lives in `packages/openclaw-core/src/`; edit there, never here.

| Shim (forwards to openclaw-core) | Core responsibility |
|------|---------|
| `index.ts` | `wakeOpenClaw()`, `initializeOpenClaw()`, `startReplyListener` / `stopReplyListener` |
| `types.ts` | `OpenClawConfig`, `OpenClawPayload`, `WakeResult` |
| `runtime-dispatch.ts` | `dispatchOpenClawEvent()`: map OpenCode events to OpenClaw events |
| `reply-listener.ts` + `reply-listener-{discord,telegram,poll-loop,process,spawn,startup,state,status,stop,signature,paths,log,sleep}.ts` | Daemon lifecycle, polling, PID/state, config signature |
| `daemon.ts` | Detached daemon entry |
| `session-registry-{storage,lock,paths,types}.ts` | Message ID to session/pane registry |

Gateway resolution + URL validation (`config.ts`), HTTP/shell dispatch (`dispatcher.ts`), `session-registry.ts`, reply injection (`reply-listener-injection.ts`), and tmux helpers (`tmux.ts`) have no shim here; import them from `@oh-my-opencode/openclaw-core/*`.

## GATEWAY TYPES

| Type | Config | Execution |
|------|--------|-----------|
| **HTTP webhook** | `url` field | POST with JSON payload |
| **Shell command** | `command` field | Execute with env vars (OPENCLAW_*) |

## PAYLOAD VARIABLES (interpolation)

`{sessionId}`, `{projectPath}`, `{tmuxSession}`, `{timestamp}`, `{eventType}` (session.created/deleted/idle), `{messageContent}`, `{promptSummary}`

## INTEGRATION POINTS

- `src/testing/create-plugin-module.ts` - calls `initializeOpenClaw(pluginConfig.openclaw)` at plugin startup (if `enabled`; injectable dep)
- `src/plugin/event-session-lifecycle.ts` - `dispatchOpenClawSessionEvent()` calls `dispatchOpenClawEvent()` for session.created/deleted/idle; `create-managers.ts` and `plugin/tool-registry-core-tools.ts` also dispatch
- `src/config/schema/openclaw.ts` - Zod config schema

## DAEMON LIFECYCLE

```
initializeOpenClaw(config)
  → wakeOpenClaw() if reply_listener.enabled
  → spawn daemon.ts as detached process
  → daemon writes PID to .opencode/openclaw.state.json
  → daemon polls Discord/Telegram every 3s
  → on reply: lookup in session-registry → inject into tmux via send-keys
```

## SECURITY

- **URL validation**: HTTPS required except localhost (openclaw-core `config.ts`)
- **Authorized users**: Inbound replies filtered by allowed user ID list
- **Token redaction**: Secrets masked in logs and error messages
- **Rate limiting**: Reply injection throttled per pane

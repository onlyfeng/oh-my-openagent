# src/openclaw/ — OpenClaw Adapter Shims (implementation in openclaw-core)

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

Every `.ts` file here (except `AGENTS.md`) is a ONE-LINE re-export shim: `export * from "@oh-my-opencode/openclaw-core/<module>"`. The harness-neutral gateway, daemon, session registry, and tmux-injection primitives live in [`packages/openclaw-core/`](../../../openclaw-core/AGENTS.md); this directory exists only to keep the OpenCode import paths (`../openclaw`, `../openclaw/reply-listener-*`, ...) stable while the implementation moves. Do NOT add logic here — extend `openclaw-core` and keep the shim. `__tests__/config.test.ts` is the only real code (config parsing contract).

## FLOW (semantics live in openclaw-core)

- Outbound: OpenCode session event -> `runtime-dispatch.ts` (maps event -> OpenClaw event) -> core `dispatcher.ts` (HTTP POST or shell command) -> core `session-registry.ts` (records message id ↔ session ↔ tmux pane).
- Inbound: Discord/Telegram API -> `reply-listener.ts` daemon (detached Bun process, `daemon.ts` entry; polls every 3s; state in `reply-listener-state.ts`) -> core session-registry lookup -> core `reply-listener-injection.ts` send-keys into the tmux pane (rate limited).
- Gateways: HTTP webhook (`url`) or shell command (`command`); payload interpolation `{sessionId}`, `{projectPath}`, `{tmuxSession}`, `{timestamp}`, `{eventType}`, `{messageContent}`, `{promptSummary}`.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Change dispatch/daemon/registry/injection behavior | `packages/openclaw-core/src/` (this dir only re-exports) |
| Startup wiring | `index.ts` shim re-exports `initializeOpenClaw()`; invoked as `deps.initializeOpenClaw(pluginConfig.openclaw)` in `src/testing/create-plugin-module.ts` (only when the openclaw config exists) |
| Event wiring | `src/plugin/event.ts` calls `dispatchOpenClawEvent()` for session.created/deleted/idle |
| Config schema | `src/config/schema/openclaw.ts` (Zod); URL validation (HTTPS except localhost) in core `config.ts` |
| Daemon state | `.opencode/openclaw.state.json` (PID, config signature); `reply-listener-status.ts` / `-stop.ts` / `-startup.ts` / `-spawn.ts` / `-poll-loop.ts` / `-sleep.ts` / `-signature.ts` / `-log.ts` / `-paths.ts` / `-process.ts` cover the daemon lifecycle shims |

## ANTI-PATTERNS

- NEVER implement behavior in a shim file; a shim's whole body is the single `export * from` line.
- Security invariants (enforced in core): HTTPS required except localhost; inbound replies filtered by authorized user id list; secrets masked in logs; per-pane rate limiting.

## COMMANDS

```bash
bun test packages/omo-opencode/src/openclaw   # config.test.ts
```

Parent: [`../AGENTS.md`](../AGENTS.md).

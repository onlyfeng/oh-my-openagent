# src/hooks/runtime-fallback/ (Reactive Provider Error Recovery)

**Generated:** 2026-07-17 (7d664b96b)

## OVERVIEW

~69 files (~15.7k LOC incl. tests; 34 implementation modules). Session Tier hook that **reactively** switches to fallback models when API providers return errors at runtime (429, 503, quota exhausted, cooldown signals). Distinct from `model-fallback`, which applies proactively at `chat.params` over hardcoded `AGENT_MODEL_REQUIREMENTS` chains with module-global `pendingModelFallbacks` state; config here comes from `categories[].fallback_models` / `agents[].fallback_models`. The two operate **independently**; no direct integration.

## ERROR DETECTION

- Default retry HTTP codes (configurable): `429, 500, 502, 503, 504`. `constants.ts` message regexes: rate limit, too many requests, quota reset, exhausted capacity, all credentials for model, cool-down, model not supported, service unavailable, overloaded, temporarily unavailable.
- `error-classifier.ts` `classifyErrorType()`: `missing_api_key`, `model_not_found`, `quota_exceeded`. `auto-retry-signal.ts` extracts "retrying in ..." style provider signals and triggers immediate fallback.

## FALLBACK CHAIN + STATE

Chain resolution (`fallback-models.ts` `getRawFallbackModelsForSession`): 1) session category `fallback_models` (SessionCategoryRegistry); 2) agent config `fallback_models`, then the agent's own category (`tryGetFallbackFromAgent`); 3) session-ID pattern match → agent → step 2. When the resolved agent is `plan` and `sisyphus_agent` is enabled (`disabled !== true`, `planner_enabled !== false`, `replace_plan !== false`), it inherits `prometheus`'s fallback models after steps 2-3; explicit `plan` `fallback_models` (even `[]`) takes precedence and suppresses inheritance.

Per-session `FallbackState` (`fallback-state.ts`): `originalModel`/`currentModel`, `fallbackIndex`, `failedModels: Map<model, cooldown-until>`, `attemptCount`, `pendingFallbackModel?`. Failed models enter 60s cooldown; `findNextAvailableFallback()` skips models in cooldown, preventing thrashing on persistently failing models.

## RETRY FLOW

```
session.error / message.updated (with error) / session.status (retry signal)
  → isRetryableError(error)?
  → getFallbackModelsForSession(sessionID, agent)
  → findNextAvailableFallback(): skip cooldown models
  → prepareFallback(): update state, mark current failed
  → dispatchFallbackRetry(): toast notification + promptAsync with new model
  → 30s timeout: abort and try next if exceeded
```

## KEY FILES

| File(s) | Purpose |
|---------|---------|
| `hook.ts` | `createRuntimeFallbackHook()`: composes all handlers |
| `event-handler.ts` / `message-update-handler.ts` / `session-status-handler.ts` / `chat-message-handler.ts` | Route session lifecycle (created, error, stop, idle); error parts in `message.updated`; retry signals in `session.status`; model override on `chat.message` |
| `error-classifier.ts` / `auto-retry-signal.ts` | `isRetryableError()`, `classifyErrorType()`; "retrying in..." signal extraction |
| `fallback-state.ts` | State machine: `createFallbackState`, `prepareFallback`, `findNextAvailableFallback`, `isModelInCooldown` |
| `fallback-models.ts` / `fallback-bootstrap-model.ts` | Chain resolution from config hierarchy (strings + raw objects); initial model when state missing |
| `fallback-retry-dispatcher.ts` | Toast + dispatch retry orchestration |
| `auto-retry-dispatch.ts` | Prompt-gate reservation, marker preservation, bounded reservation backoff, stale-generation rollback |
| `auto-retry-timeout.ts` / `auto-retry.ts` | `createFallbackTimeoutHelpers` - timeout-driven retry scheduling + expiry actions; abort, timeout scheduling, cleanup |
| `auto-retry-metadata.ts` / `last-user-retry-parts.ts` | `RetryPromptPart`, `resolveOriginalUserRetryMetadata`; extract last user message parts for retry |
| `auto-retry-abort.ts` / `auto-retry-agent-context.ts` / `agent-resolver.ts` | `createAbortSessionRequest`, `createAgentContextResolver`; session → agent name normalization |
| `auto-retry-cleanup.ts` / `stale-session-cleanup.ts` | Lazy fallback-state clearing for dead sessions |
| `fallback-state-snapshot.ts` | `snapshotFallbackState` / `restoreFallbackState` |
| `normalize-model.ts` / `runtime-model-settings.ts` / `retry-model-payload.ts` | `normalizeModelToCanonicalString`; `RuntimeModelSettings` + `resolveRuntimeModelSettings`; model payload (providerID/modelID/variant/reasoningEffort) |
| `session-messages.ts` / `visible-assistant-response.ts` | Session message part shapes shared by handlers; real output vs only-errors detection (prevents retry on partial valid response) |
| `first-prompt-watchdog.ts` | Timeout fallback when a session never produces its first prompt response (`first_prompt_timeout_seconds`, 300s default, 0 disables) |

## NOTES

- Cooldown and failure tracking are **per-session**; concurrent sessions don't share state
- `visible-assistant-response.ts` prevents retry if the assistant already produced a partial valid response
- Runtime-fallback is registered in the Session Tier via `create-session-hooks.ts`

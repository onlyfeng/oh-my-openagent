# src/hooks/anthropic-context-window-limit-recovery/ (Multi-Strategy Context Recovery)

**Generated:** 2026-05-15 · **Updated:** 2026-11-18 / c2691818a9

## OVERVIEW

49 TS files + this doc (~5.5k LOC incl. tests; `storage/` subdir persists tool results). Most complex hook in `src/hooks/`. Recovers from context window limit errors via multiple strategies applied in sequence. Entry `index.ts` exports `createAnthropicContextWindowLimitRecoveryHook(ctx, options)` with an injectable-dependency seam (`executeCompact`, `getLastAssistant`, `log`, `parseAnthropicTokenLimitError`) used by tests.

## RECOVERY STRATEGIES (in priority order)

| Strategy | File | Mechanism |
|----------|------|-----------|
| **Empty content recovery** | `empty-content-recovery.ts` | Handle empty/null content blocks in messages |
| **Deduplication** | `deduplication-recovery.ts` | Remove duplicate tool results from context |
| **Target-token truncation** | `target-token-truncation.ts` | Truncate largest tool outputs to fit target ratio |
| **Aggressive truncation** | `aggressive-truncation-strategy.ts` | Last-resort truncation with minimal output preservation |
| **Summarize retry** | `summarize-retry-strategy.ts` | Compaction + summarization then retry |

## KEY FILES

| File | Purpose |
|------|---------|
| `recovery-hook.ts` | Main hook entry: `session.error` handler, strategy orchestration |
| `executor.ts` | Execute recovery strategies in sequence |
| `parser.ts` | Parse Anthropic token limit error messages |
| `state.ts` | `AutoCompactState`: per-session retry/truncation tracking |
| `types.ts` | `ParsedTokenLimitError`, `RetryState`, `TruncateState`, config constants |
| `storage.ts` | Persist tool results for later truncation |
| `tool-result-storage.ts` | Store/retrieve individual tool call results |
| `message-builder.ts` | Build retry messages after recovery |
| `client.ts` | `Client` type the hook needs beyond `PluginInput`: `session.promptAsync` + `tui.showToast` |
| `session-timeout-map.ts` | `clearSessionTimeout` / `clearAllSessionTimeouts` — per-session timers must be cleared (300 ms debounce before compaction) |
| `message-storage-directory.ts` | `getMessageIds(sessionID)` over the shared `getMessageDir` |
| `storage-paths.ts` | `MESSAGE_STORAGE_DIR` / `PART_STORAGE_DIR` re-exports + `TRUNCATION_MESSAGE` |
| `tool-part-types.ts` | `StoredToolPart`, `ToolResultInfo` shapes |

## RETRY CONFIG

- Max attempts: 2
- Initial delay: 2s, backoff ×2, max 30s
- Max truncation attempts: 20
- Target token ratio: 0.5 (truncate to 50% of limit)
- Chars per token estimate: 4
- Min output size eligible for truncation: 500 chars (`minOutputSizeToTruncate`)

## PRUNING SYSTEM

`pruning-*.ts` files handle intelligent output pruning:
- `pruning-deduplication.ts`: Remove duplicate content across tool results
- `pruning-tool-output-truncation.ts`: Truncate oversized tool outputs
- `pruning-types.ts`: Pruning-specific type definitions

## SDK VARIANTS

`empty-content-recovery-sdk.ts` and `tool-result-storage-sdk.ts` provide SDK-based implementations for OpenCode client interactions.

## STORAGE SUBDIR (`storage/`, 11 files)

Readers/writers over OpenCode's on-disk message/part storage: `messages-reader.ts` + `parts-reader.ts` (stream entries), `message-dir.ts`, `part-id.ts` / `part-content.ts`, `text-part-injector.ts`, `empty-messages.ts` / `empty-text.ts` (empty-content recovery primitives), `constants.ts`, `types.ts` (`StoredMessageMeta`, `StoredTextPart`, `StoredToolPart`); `readers-from-sdk.test.ts` pins the SDK parity.

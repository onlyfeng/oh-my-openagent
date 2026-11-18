# session-manager - the 4 `session_*` tools

**Score 8** (18 files, ~4.0k LOC; distinct domain: the only tool family with dual SDK/file storage and fallback merge).

## OVERVIEW

Implements `session_list`, `session_read`, `session_search`, `session_info` (tool catalog + registration gates live in the parent [`../AGENTS.md`](../AGENTS.md)). `tools.ts` builds the `ToolDefinition`s via `createSessionManagerTools(input)`; the rest of the directory is storage selection, formatting, and error taxonomy.

## ARCHITECTURE

- **Dual storage, merged by session ID** (`storage.ts`): SQLite via the OpenCode SDK client (`sdk-storage.ts`, paged reads at `SDK_MESSAGE_PAGE_SIZE`) when `isSqliteBackend()`; legacy JSON storage (`file-storage.ts`) for compatibility and fallback. Session lists merge by ID, newest first.
- **Fallback rule:** an SDK read falls through to file storage when `shouldFallbackFromSdkError(error)` OR the SDK answers `NotFoundError` - a session existing only in legacy JSON storage (#5854) must still be readable.
- **Project filter:** `ctx.directory === "/"` (multi-project `opencode serve` / `opencode web`) normalizes to no filter; otherwise every session would be silently dropped.
- **Bounded work:** `session_search` scans at most `MAX_SESSIONS_TO_SCAN = 50` sessions under `SEARCH_TIMEOUT_MS = 60s`; `runWithAbortableTimeout` aborts the underlying reads on timeout (composed with the caller's abort signal) instead of letting them run on.
- **Formatting/errors:** `session-formatter.ts` renders list/read/search/info payloads; `session-error.ts` (`describeSessionError`, `SkippedSession` → `formatSkippedSessions`) is the error taxonomy.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Tool definitions + argument schemas | `tools.ts`, `types.ts` (`SESSION_*_DESCRIPTION` in `constants.ts`) |
| Storage selection / merge / fallback | `storage.ts` |
| SDK (SQLite) reads + message paging | `sdk-storage.ts` |
| Legacy JSON reads | `file-storage.ts` |
| Output formatting | `session-formatter.ts` |
| Global server client + basic-auth header | `createGlobalSessionSdkClient` in `tools.ts` (`getServerBasicAuthHeader` from `../../shared`) |

## CONVENTIONS

- Client injection: `setStorageClient()` swaps the SDK client for tests; `sdk-unavailable.ts` covers the no-SDK path.
- Behavior is pinned by colocated suites: `storage.test.ts` (790 LOC) + `storage-fallback.test.ts` (511 LOC) own the merge/fallback matrix; `session-paging.test.ts` the bounded windows.

## COMMANDS

- `bun test packages/omo-opencode/src/tools/session-manager` (focused)
- Typecheck: `bun run --cwd packages/omo-opencode typecheck`

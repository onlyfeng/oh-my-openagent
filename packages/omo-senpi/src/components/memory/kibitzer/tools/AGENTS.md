# memory/kibitzer/tools/ — Kibitzer Sidecar Tool Surface

**Score ~14** (16 files, ~2.1k LOC; closed member-scoped tool registry).

## OVERVIEW

The resident Kibitzer sidecar's read-only tool surface. Every tool is a member-scoped closure (never a builtin passthrough) so only closures can enforce output caps, the per-wake budget, and redaction. `index.ts` composes exactly five tools and exports the registry contract; `sidecar-prompt-blocks.ts` holds the parallel prompt-side copy and tests assert both lists match.

## THE FIVE TOOLS (closed set — no aliases, no writes)

| Tool | Factory | File |
|------|---------|------|
| `read` | `createKibitzerReadTool` | `read.ts` |
| `grep` | `createKibitzerGrepTool` | `grep.ts` (~269 LOC — most complex: bounded walk + scan budget) |
| `session_entries` | `createKibitzerSessionEntriesTool` | `session-read.ts` |
| `memory` | `createKibitzerMemoryTool` | `memory.ts` |
| `nudge` | `createKibitzerSidecarNudgeTool` | `nudge.ts` |

Support: `index.ts` (registry + `KIBITZER_SIDECAR_TOOL_NAMES`), `result.ts` (`okText`/`okJson`/`rejection`/`boundedText`/`budgeted`), `caps.ts` (`DEFAULT_KIBITZER_TOOL_CAPS`), `budget.ts` (`createWakeToolBudget`), `path-safety.ts` (`normalizeMemoryPath`, `resolveWorkspacePath`).

## CONVENTIONS

- Uniform result shaping: success via `okText`/`okJson`; refusals via `rejection(code, message, path)` → structured JSON `isError: true`, never thrown.
- Redact FIRST, then truncate (`boundedText`) — a secret split by truncation would escape the pattern.
- Caps/budget are internal (composition-tunable only via `resolveKibitzerToolCaps`); each wake gets one `createWakeToolBudget(limit)` charged via `budgeted()`.
- Paths are fail-closed: memory paths are POSIX repo-relative (reject empty/absolute/`\\`/`..`/`system/`); workspace paths resolve through `realpath` on both sides.

## ANTI-PATTERNS

- No builtin passthrough, no aliases, no write capability of any kind in the registry.
- The sidecar must stay read-only; MEMORY reads come from HEAD, never the working tree.
- Never second-guess an explicitly named grep file; the walk never follows symlinks and skips VCS/dependency trees.

## COMMANDS

- `bun test packages/omo-senpi` · `tsgo --noEmit -p packages/omo-senpi/tsconfig.json` · `bun run test:senpi`

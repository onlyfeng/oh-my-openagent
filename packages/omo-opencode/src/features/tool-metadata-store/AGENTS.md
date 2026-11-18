# tool-metadata-store - Cross-Hook Tool Metadata Handoff

**Score:** 10 (12 files, 13 barrel exports, ~18 consumer files across omo-opencode hooks; distinct domain: surviving OpenCode's tool-metadata overwrite). Parent: [`../AGENTS.md`](../AGENTS.md) carries a one-row summary only.

## OVERVIEW

Pending store + task-metadata contract that lets a tool execution hand structured metadata to later hooks. Exists because OpenCode's `fromPlugin()` wrapper ALWAYS overwrites a plugin tool's returned metadata with `{ truncated, outputPath }`, discarding anything the tool set during `execute()` (see the `store.ts` header).

## WHERE TO LOOK

| File | Role |
|------|------|
| `store.ts` | Pending store: `storeToolMetadata` / `consumeToolMetadata` / `getPendingStoreSize` / `clearPendingStore` |
| `resolve-tool-call-id.ts` | `resolveToolCallID` + `ToolCallIDCarrier` - one id across the execute/after boundary |
| `task-metadata-contract.ts` | `buildTaskMetadataBlock` / `parseTaskMetadataBlock` / `extractTaskLink` + `TaskLink` - machine-consumed block format |
| `publish-tool-metadata.ts` | `publishToolMetadata` - store during `execute()` |
| `recover-tool-metadata.ts` | `recoverToolMetadata` - restore in `tool.execute.after` |
| `index.ts` | Barrel (all of the above) |

## CONSUMERS

The `tool.execute.after` merge path plus metadata/link readers: atlas, ralph-loop, task-resume-info, background-task (~18 files).

## ANTI-PATTERNS

- NEVER rely on metadata set during `execute()` surviving `fromPlugin()` - always go through the store + `tool.execute.after` merge.
- The metadata block is a parsed contract (`parseTaskMetadataBlock`/`extractTaskLink`); consumers assert on parsed fields, never on prose.

## COMMANDS

```bash
bun test packages/omo-opencode/src/features/tool-metadata-store/
```

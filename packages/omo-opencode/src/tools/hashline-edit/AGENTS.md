# src/tools/hashline-edit/ - Hash-Anchored `edit` Tool

**Generated:** 2026-11-18 | **Source snapshot:** c2691818a91020e6d9d8ffb513bedca364ebceee | **Branch:** local/agents-md-refresh

## OVERVIEW

24 files. Hash-anchored editing: every line reference carries a content hash (`LINE#ID`), validated before edits apply, so stale references are rejected instead of corrupting the file. Registered as the `edit` tool, gated on `hashline_edit: true` (see parent [`tools/AGENTS.md`](../AGENTS.md)), replacing OpenCode's line-number edit.

## SPLIT: LOCAL CODE vs HASHLINE-CORE SHIMS

Real code here (4 files):

| File | Purpose |
|------|---------|
| `hashline-edit-executor.ts` | Orchestrator (~158 LOC): canonicalize → normalize → validate/apply via shims → diff → formatter-trigger; publishes tool metadata via `../../features/tool-metadata-store` |
| `tools.ts` | `createHashlineEditTool()` factory - tool schema + entry point |
| `tool-description.ts` | `HASHLINE_EDIT_DESCRIPTION` constant (documents LINE#ID + alphabet `ZPMQVRWSNKTXJBYH`) |
| `formatter-trigger.ts` | `runFormattersForFile()` / `FormatterClient` - post-edit formatter run (~118 LOC) |

Every other `.ts` file is a shim re-exporting [`@oh-my-opencode/hashline-core`](../../../../hashline-core/AGENTS.md): `normalize-edits.ts` (`normalizeHashlineEdits`), `validation.ts` (`HashlineMismatchError`), `edit-ordering.ts` (`getEditLineNumber`, `collectLineRefs`, `detectOverlappingRanges`), `edit-deduplication.ts` (`dedupeEdits`), `edit-operations.ts` (`applyHashlineEditsWithReport`), `edit-operation-primitives.ts`, `edit-text-normalization.ts`, `file-text-canonicalization.ts` (`canonicalizeFileText`/`restoreFileText`), `autocorrect-replacement-lines.ts`, `hashline-edit-diff.ts` (`generateHashlineDiff`), `diff-utils.ts` (`generateUnifiedDiff`, `countLineDiffs`), `hashline-chunk-formatter.ts`, `hash-computation.ts`, `constants.ts`, `types.ts` (`ReplaceEdit`/`AppendEdit`/`PrependEdit`/`HashlineEdit`), `index.ts` (barrel).

Do NOT add logic to a shim file - extend `hashline-core` and keep the shim.

## THREE-OP MODEL

| Op | pos | end | lines | Effect |
|----|-----|-----|-------|--------|
| `replace` | required | optional | required | Replace line or range pos..end |
| `append` | optional | optional | required | Insert after anchor (or EOF) |
| `prepend` | optional | optional | required | Insert before anchor (or BOF) |

`lines: null` / `[]` with `replace` = delete lines; `delete: true` at tool level = delete file.

## LINE#ID FORMAT

`{line_number}#{hash_id}`; hash_id is 2 chars from `ZPMQVRWSNKTXJBYH`. Read output separates tag and content with `|` (`42#VK| function hello() {`). Validation recomputes the current line's hash; mismatch → rejected with a diff of current state.

## ERROR CASES

- Hash mismatch → rejected + current-state diff (`HashlineMismatchError`)
- Overlapping ranges → rejected (`detectOverlappingRanges`)
- Missing `pos` for `replace` → schema error
- `lines: null` with `append`/`prepend` → schema error

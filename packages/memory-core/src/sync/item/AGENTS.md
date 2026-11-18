# sync/item - item document sync domain

**Score 13** (24 impl + 23 test modules; distinct sub-domain of `src/sync/`: per-item capture, three-way merge, device-peer sync). Parent covers the sync layer's redaction rules and memory-core's harness-neutrality - not repeated.

## OVERVIEW

Item-level sync engine inside memory-core's remote-mirror stack: item documents (frontmatter + body + history), device captures with receipts, Loro-based three-way merges, key normalization, deterministic on-disk rendering, and device identity. Barrel `index.ts`; `machine-id.ts` is additionally published as the package subpath `./hardware-id`.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Item document model | `document.ts` (`itemFrontier`, `deleteItem`, `isLive`), `frontmatter.ts` (`splitItem`), `history.ts`, `state.ts`, `tag.ts` |
| Device capture | `capture.ts` (+ `CaptureBaselineError`), `capture-receipts.ts` (`readReceipts`/`writeReceipts`/`receiptLanded`), `device-capture.ts` (`captureOnDevice`, `recordMaterialized`), `device.ts`/`device-peer.ts` (device open/create, `DevicePeerError`) |
| Merge | `three-way.ts` (Loro three-way: base = one insert, sides = whole-line diffs, deterministic #12279), `transplant.ts` (replays a merged capture onto the device's own fork under its stable peer; throwaway ops never leave capture), `field-delta.ts`, `line-update.ts` |
| Keys / adoption | `key.ts` (`normalizeItemKey`, `isSyncItemKey`, `SyncKeyProblem`), `key-writers.ts`, `adopt-values.ts` (active registers) |
| Rendering | `render.ts` (`renderItem`/`renderDocContent`), `render-for-disk.ts` (deterministic disk bytes; `.r6` scenario + property tests), `kind-rule.ts` |
| Case folding | `casefold.ts` over generated `casefold-data.d.ts` + vendored Unicode `CaseFolding.txt` |
| Device identity | `machine-id.ts` - hashes OS machine id AND canonical state-dir path: a state dir copied to another machine/path mints a new peer, a same-path backup keeps it (review r4 MEDIUM / r5 M3) |

## CONVENTIONS

- Historical review scenarios keep `.r2`/`.r3`/`.r6` test suffixes (`capture.r3.test.ts`, `render-for-disk.r6.test.ts`); merge/render/key invariants also carry property tests (`key-sync.property.test.ts`, `render-for-disk.property.test.ts`).
- `test-support.ts` is the shared fixture module; `tombstone.test.ts` pins tombstone semantics; `machine-id.test.ts` pins the peer-stability contract.
- Harness-neutral like all of memory-core: no Senpi/OpenCode adapter imports; redaction happens in the parent sync layer before anything leaves the repository.

## ANTI-PATTERNS

- Never persist throwaway fork ops: capture's merge experiments (transplant) run on throwaway forks/copies and are replayed as fresh ops under the device's stable peer - leaking them would corrupt the device's own fork history.
- Do not derive device identity from the OS machine id alone (`machine-id.ts` includes the canonical state-dir path by design).

## COMMANDS

- From `packages/memory-core`: `bun run typecheck`; `bun run test` (`bun test src/`); focused `bun test src/sync/item/`.

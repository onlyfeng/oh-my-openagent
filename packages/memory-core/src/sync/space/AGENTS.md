# src/sync/space/ -- Sync Space Model

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 11 (26 files, dense CRDT-sync domain, own barrel).

## OVERVIEW

The space model of memory sync (sync-design §3 / #454 §3): which sync space a repository belongs to, and how text with no shared CRDT history joins a space without losing a character. Pure planning over injected stores (`SpaceConfigStore`, `LoroDoc`); git runs only through `GitSpaceConfig` on the repo's LOCAL config.

## WHERE TO LOOK

| File | Role |
|------|------|
| `binding.ts` | BindingKey kinds `named:` / `repo:` / `default:` / legacy `local:`; `ACCOUNT_DEFAULT_BINDING` = `default:account`; `canonicalRemoteUrl`, `spaceIdForBinding` (sha-256 truncated to 128 bits), `syncedAgentDirName` |
| `space.ts` | `resolveSpace()`: binding key + sticky `omo.sync.space` record (recorded id wins over a changed remote URL); `GitSpaceConfig`, `repoHasCommit`, `listRemotes`, `resolveSpaceInRepo`; legacy `local:` spaces migrate to the account default (#13720) and the legacy id is reported every resolve — the binding is never rewritten |
| `adopt.ts` | `planAdoption()` → insert-only `AdoptionPlan` (`none` / `remove` / `capture` + `workingText` + `extraRegisterValues`); a file identical to `lastMaterialised` is skipped; local-deletion shortcut is an ordered-subsequence proof |
| `adopt-frontmatter.ts` | Register/set field adoption per key; `seedFields` throws unless `registers` holds each register key's exact components (a rendered register is never read as one whole value) |
| `read-back.ts` | `firstMissingFact()`: structure-free deletion gate — every source line held ≥ as many times, key lines counted only via active register entries / set elements |
| `confirm-removals.ts` | Deletion gate on the COMMITTED doc; engine order binding: (1) capture, (2) import + persist durably, (3) `confirmRemovals`, (4) delete confirmed sources |
| `merge.ts` | `mergeInsertions()`: common supersequence; lines are indivisible; alignment runs under a cell budget, beyond which complete lines concatenate |
| `hub-trace.ts` | Exactly two hub shapes: T1 `<stem>.sync-conflict-…` side files, T2 md-merge-driver pickFrontmatter body blocks |
| `manifest.ts` | Per-space Loro manifest: add-wins binding set, display name, `absorbed_into` tombstone |
| `join.ts` | `joinSpaces()`: two spaces sharing a binding key join toward the LOWER space id; tombstone makes the join idempotent |
| `index.ts` | Public barrel: adopt / binding / confirm-removals / hub-trace / join / manifest / merge / space (`adopt-frontmatter`, `read-back`, `test-support` stay internal) |

## CONVENTIONS

- Deletion is never silent and never planner-predicted: a source is removed only after the read-back on the prediction AND `confirmRemovals` on the committed doc; anything else stays on disk with a structured reason in `retainedSources`.
- No marker or conflict text is ever written into an item body.
- Git never runs with the inherited environment — `memoryGitEnvironment` strips `GIT_DIR`/`GIT_WORK_TREE` so another repo's config cannot leak (#12354).

## ANTI-PATTERNS

- NEVER splice similar-looking prose character by character; similar text is not evidence of shared history.
- NEVER delete a source before step (2) is durable — a crash would lose the only copy.
- NEVER admit a superseded register value through a side file: the read rule is ACTIVE entries only (a snapshot cannot tell a revert from a stale replay).

## QA

`bun test --timeout 20000 packages/memory-core/src/sync/space/` — every module has co-located tests (adopt, read-back, join, manifest, merge, hub-trace, binding).

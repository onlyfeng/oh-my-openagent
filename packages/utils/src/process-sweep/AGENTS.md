# process-sweep — Family-Based Orphan Process Cleanup

**Score:** 13 (11 files, 1.3k LOC, index barrel, two adapter consumers). Parent [`packages/utils/AGENTS.md`](../../AGENTS.md) states the package-level invariant: the LSP daemon SERVER shape (`cli.js daemon`, no `mcp` arg) is NEVER a proxy candidate — `lsp-proxy-family.ts` owns that rule.

## OVERVIEW

Enumerates processes, groups them into families, and sweeps orphans: stale LSP daemon proxy processes, superseded daemon versions, and unrelated leftovers under omo-owned roots.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Barrel | `index.ts` |
| Enumeration + kill | `exec.ts` — `enumerateProcesses`, `createDefaultProcessKiller`, `defaultIsProcessAlive` |
| Process table + orphaning | `process-table.ts` — `isOrphaned`, `parsePosixProcessTable`, `parseWindowsProcessTable` |
| Family sweeps | `family-sweeper.ts`, `sweeper.ts` — `sweepOrphanedLspDaemonProxies`, `sweepStaleLspDaemonVersions` |
| LSP daemon discovery | `lsp-daemon-family.ts` (stale-version planning), `lsp-proxy-family.ts` (proxy candidate matching) |
| Ownership | `lsp-daemon-owner-attestation.ts` (owner attestation), `roots.ts` — `discoverOmoOwnedRoots` |
| Command matching | `command-match.ts` |

## CONVENTIONS

- Consumers: `packages/omo-opencode/src/shared/omo-process-sweep.ts` and the senpi-task process sweep — both go through this barrel.
- Platform parsing is split (`parsePosixProcessTable` / `parseWindowsProcessTable`); add platform-specific handling there, not at call sites.
- Kill operations go through the injected `ProcessKiller`; tests fake it.

## ANTI-PATTERNS

- Never match the daemon server shape as a sweep candidate (see parent invariant; tests pin it).
- Never kill by name alone — family + ownership attestation decide.

## COMMANDS

```bash
bun test --timeout 20000 packages/utils/src/process-sweep
bun run --cwd packages/utils typecheck
```

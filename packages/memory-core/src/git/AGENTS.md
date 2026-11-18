# git/ - Git Subprocess Boundary for the Memory Repository

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 14 (74 files, own barrel, `GitMemoryRepo` at the center of every memory write). Parent [`../../AGENTS.md`](../../AGENTS.md) owns the memory invariants (harness-neutral, transactional writes, lock domains); this file maps the boundary's modules.

## STRUCTURE

| Area | Files | Role |
|------|-------|------|
| Barrel | `index.ts` | Re-exports `config-lock`, `errors`, `exec`, `path-state`, `porcelain`, `repo`, `repo-lock-unlock`, `secret` |
| Facade | `repo.ts` | `GitMemoryRepo` - initialization, clean checks, commits, revisions, merge worktrees, remote detection |
| Repo satellites | `repo-head.ts` (HEAD reads incl. reftable), `repo-revision-reads.ts`, `repo-log.ts` / `repo-tree.ts` / `repo-status.ts` (bounded reads over `porcelain.ts`), `repo-commit-times.ts`, `repo-line-endings.ts`, `repo-unicode-paths.ts`, `repo-in-use.ts`, `repo-maintenance.ts`, `repo-config-defaults.ts`, `repo-secret-staging.ts`, `repo-arguments.ts`, `repo-types.ts` | One concern each, all executing through the facade's `GitExec` |
| Exec boundary | `exec.ts` (injectable `GitExec` + `createNodeGitExec`), `exec-environment.ts`, `timeout-diagnostics.ts`, `local-filesystem.ts` | Every git spawn goes through `GitExec`; env, timeout, pipe-holder, stdin-EPIPE, fsmonitor, termination, and Windows-console behavior each have a dedicated test file |
| Path state | `path-state.ts`, `path-state-files.ts`, `path-state-index.ts` | Worktree/path cleanliness state over the tree |
| Locks | `git-activity-lock.ts`, `config-lock.ts`, `lock-scans.ts` | Git never runs unseen: the activity lock is acquired or the operation refuses |
| Stranded locks | `repo-lock-sweep.ts` (+5 focused suites), `repo-lock-unlock.ts`, `open-file-holders.ts` (slow-lsof holder discovery), `proc-view.ts` / `proc-scan*.ts` (bounded process inspection) | Reclaim only with proven-dead ownership; `inspectStrandedGitLock` / `unlockStrandedGitLock` refuse on any visible live holder |
| Rollback | `pending-rollback.ts`, `worktree-restore.ts`, `worktree-mutation-queue.ts` | Serialized worktree mutation + restore |
| Misc | `secret.ts` (secret screening), `no-auto-maintenance.ts`, `stable-log-output.ts`, `errors.ts` (typed git errors) | |

## CONVENTIONS

- Kebab-case modules, named exports; tests colocated as `*.test.ts`, shared harnesses as `*.test-support.ts` (`settle-within`, `test-repo-config`).
- Subprocess facts (env, timeout, Windows console, EPIPE, fsmonitor) each own a test file - extend the matching one instead of adding ad-hoc spawns.
- Unicode paths and CRLF/LF line endings have dedicated suites; new repo operations must keep both passing.

## ANTI-PATTERNS

- Do not spawn `git` directly anywhere in memory-core; go through the injected `GitExec` (tests fake it; production uses `createNodeGitExec`).
- Do not retry a genuine git error away, and do not run git when the activity lock cannot be acquired.
- Do not treat an uncertain lock/process-ownership answer as proof of safety - the sweep/unlock suites pin refusal behavior.

## COMMANDS

```bash
bun run --cwd packages/memory-core typecheck
bun test --timeout 20000 packages/memory-core/src/git
```

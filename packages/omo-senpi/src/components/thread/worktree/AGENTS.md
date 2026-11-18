# worktree — thread_create's Worktree Option

**Score:** 12 (11 files, 1.3k LOC, injectable seams, RPC extension). Parent [`thread/AGENTS.md`](../AGENTS.md): "preconditions checked before `git worktree add`, the `omo.thread.worktree` extension RPC for the remote leg, failures mapped to existing codes" (#13408).

## OVERVIEW

Creates and reaps the per-thread git worktree for `thread_create { worktree }`: local creation with typed failures and cleanup, plus the remote leg over the `omo.thread.worktree` extension RPC.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Local create | `thread-worktree.ts` — `createThreadWorktree` (`WorktreeRequest`, `ThreadWorktree`, `WorktreeOutcome`), `checkFolder`, `runGit` |
| Remote leg | `remote-worktree.ts` — `prepareRemoteWorktree`, `removeRemoteWorktree` |
| RPC surface | `worktree-rpc.ts` (`THREAD_WORKTREE_RPC`), `worktree-rpc-name.ts` |
| Teardown | `discard-worktree.ts`, `worktree-cleanup.test.ts` (336 LOC) |
| Failure mapping | `failures.ts` |
| Test kit | `git-repo.testkit.ts` (deterministic temp repos) |

## CONVENTIONS

- `GitRun` and clock are injected; tests drive both with fakes plus `git-repo.testkit.ts` temp repos — no real user checkouts.
- Validate ref/base/path BEFORE `git worktree add`; a failed add runs the cleanup path before returning the typed failure.
- Remote calls use a short-lived session and the shared close ladder; error normalization happens in `remote-worktree.ts`, not at call sites.

## ANTI-PATTERNS

- Never leave a partially created worktree behind on failure — cleanup runs even when individual steps error.
- Never `if remote` branch at tool level; the remote leg is chosen here through the RPC, not by the caller.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-senpi/src/components/thread/worktree
tsgo --noEmit -p packages/omo-senpi/tsconfig.json
```

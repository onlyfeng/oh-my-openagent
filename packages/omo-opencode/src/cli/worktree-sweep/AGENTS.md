# worktree-sweep - Stale Git Worktree Sweeper

**Score:** 12 (11 files, 1.2k LOC, index barrel, own CLI semantics). Parent [`src/cli/AGENTS.md`](../AGENTS.md) documents the command row and the default exclude prefixes (`~/.codex/worktrees`, `~/.codex-gui-cli-remote/worktrees`).

## OVERVIEW

Classifies and sweeps stale linked worktrees for the `omo worktree-sweep` command: parse `git worktree list`, classify each entry, format the report, and delete only what the options allow.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Classification + excludes | `classify.ts` - `classifyWorktree`, `DEFAULT_EXCLUDE_PREFIXES`, `isExcludedPath` |
| List parsing | `parse-worktree-list.ts` - `parseWorktreeList`, `linkedWorktrees` |
| Option parsing | `options.ts` - `parseOlderThanDays` (non-negative integer, else throws) |
| Sweep execution | `sweep.ts` - `sweepWorktrees` |
| CLI adapter | `worktree-sweep.ts` - `worktreeSweep` (command entry) |
| Output | `format.ts` - `formatClassification`, `formatResult`, `formatSummary` |
| Git access + types | `git.ts`, `types.ts`; barrel `index.ts` |

## CONVENTIONS

- Dry-run is the default: only `apply: true` removes worktrees; a run without it reports only.
- `--older-than 0` is valid and means merged-only (no staleness age filter).
- `--json` is an explicit option; human output is the default.
- Git runs through `git.ts` only, keeping parsing pure and testable (`worktree-sweep.test.ts` 627 LOC, `command-registration.test.ts` pins the CLI wiring).

## ANTI-PATTERNS

- Never sweep a worktree whose path matches an exclude prefix - external tools own those roots.
- Never delete in the same pass that reports; classification and removal stay separable for the dry-run contract.

## COMMANDS

```bash
bun test --timeout 20000 packages/omo-opencode/src/cli/worktree-sweep
bun run --cwd packages/omo-opencode typecheck
```

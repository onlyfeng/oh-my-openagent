# git-worktree — Pure Git Status/Diff Parsing

**Score:** 12 (16 files, ~1k LOC, index barrel, consumed by worktree-sweep and team tools). Parent [`packages/utils/AGENTS.md`](../../AGENTS.md) lists the group; this file maps it.

## OVERVIEW

Pure parsers over `git status --porcelain` and `git diff --numstat` output plus diff-stat collection and change formatting. No module here spawns git — callers capture output and feed it in.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Status parsing (line + batch) | `parse-status-porcelain-line.ts`, `parse-status-porcelain.ts`, `parse-status-porcelain-z.ts` |
| Diff numstat (line + NUL) | `parse-diff-numstat.ts`, `parse-diff-numstat-z.ts` |
| Diff-stat collection | `collect-git-diff-stats.ts` (+ `collect-git-diff-stats.real-git.test.ts`, real-git integration) |
| Change formatting | `format-file-changes.ts` |
| Types | `types.ts` (`GitFileStatus`, `GitFileStat`); barrel `index.ts` |

## CONVENTIONS

- Every parser has a line-oriented AND a NUL-delimited (`-z`) variant — extend both when git output handling changes.
- Parsing is total: malformed input yields typed errors or skips, never throws on adversarial filenames (rename/copy entries with arrows, quoted paths).
- Consumers: `packages/omo-opencode/src/cli/worktree-sweep/` (classification) and team-mode worktree reporting.

## ANTI-PATTERNS

- Never spawn git from this directory — it is the pure half by design.
- Never parse porcelain output with naive whitespace splits; quoted/non-ASCII paths are covered by the colocated tests (largest: `collect-git-diff-stats.real-git.test.ts`, 197 LOC).

## COMMANDS

```bash
bun test --timeout 20000 packages/utils/src/git-worktree
bun run --cwd packages/utils typecheck
```

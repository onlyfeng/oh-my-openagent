# git - Side-Panel Git Data Pipeline

**Generated:** 2026-11-18 / c2691818a9

**Score:** 10 (8 files, ~1.1k LOC; no `side-panel/AGENTS.md` exists - this file carries the git section's data layer; rendering lives in the side-panel component).

## OVERVIEW

Pure data pipeline for the side panel's git section: locate the repo root, read status through an injected exec seam, parse NUL-separated porcelain, build colored diff rows. The module never spawns a process of its own.

## PIPELINE

| Stage | File | Exports |
|-------|------|---------|
| Root + branch | `repo.ts` | `findGitRoot`, `readGitBranch` |
| Status read | `read.ts` | `readGitStatus(exec, root)` -> `PanelGitStatus`; `PanelExec` / `PanelExecResult` seam |
| Parse + merge | `parse.ts` | `parsePorcelainZ`, `parseNumstatZ`, `mergeGitDeltas`, `PanelGitEntry`, `PanelGitDelta` |
| Diff rows | `diff.ts` | `gitDiffArgs`, `readGitDiff`, `colorizeDiff`, `UNTRACKED_DIRECTORY_FILE_CAP = 50` |

## CONVENTIONS

- Git output is parsed NUL-separated (`--porcelain -z`, `--numstat -z`) - never line-split filenames.
- Untracked directory expansion is capped (`UNTRACKED_DIRECTORY_FILE_CAP`).
- Every function is pure over its `PanelExec` input; tests run on fixture output, no repo required.

## ANTI-PATTERNS

- Do not call `node:child_process` from here - the exec seam is the component's testability contract.
- Do not widen the untracked cap without its test.

## GOTCHAS (source-verified)

- `readGitStatus` fires three parallel reads (`status --porcelain=v1 -z --untracked-files=normal`, `diff --numstat -z`, `diff --cached --numstat -z`); a failed status read returns `undefined` - a partial answer would misreport a clean tree.
- Renames need BOTH paths in the pathspec (`diff HEAD -M -- <from> <path>`); the destination alone prints the whole file as additions.
- Untracked files diff via `--no-index` against `/dev/null`, which exits 1 by design when files differ - exit 1 is success here; exit 128 with empty stdout on a staged file triggers a `--no-index` retry (path no longer in HEAD).
- Paths carry per-pathspec `:(literal)` magic, never the global `--literal-pathspecs` flag (it conflicts with exported `GIT_GLOB_PATHSPECS`/`GIT_ICASE_PATHSPECS` - git exits 128, #11176); the magic is skipped when `GIT_LITERAL_PATHSPECS` is already truthy.
- `repo.ts` never spawns a process: `.git` may be a file (worktree) whose `gitdir:` pointer is followed once; detached HEAD yields a 7-char sha.
- Untracked directories: one `ls-files --others --exclude-standard -z` listing, then one diff process per file up to the cap; the remainder is counted in a `... N more files` row.

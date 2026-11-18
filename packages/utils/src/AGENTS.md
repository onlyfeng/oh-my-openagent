# src — Flat Utility Files over Domain Subdirs (direct files)

**Generated:** 2026-11-18 / c2691818a9

**Score:** 29 (70 direct files, 9 subdirs, package barrel + public subpaths). WHAT each utility does lives in the parent [`packages/utils/AGENTS.md`](../AGENTS.md) category map — this file covers only the directory's own layout doctrine.

## OVERVIEW

Single-purpose utilities ship as flat `*.ts` with colocated `*.test.ts`; a domain that outgrows one file becomes a subdir with its own `index.ts` barrel. `src/index.ts` is the package barrel.

## SUBDIRS (9)

`ast-grep/`, `command-executor/`, `git-worktree/`, `logging/`, `migration/`, `process-sweep/`, `prompt-async-gate/`, `runtime/`, `zip-entry-listing/`. Three carried their own AGENTS.md before this pass (`ast-grep/`, `git-worktree/`, `process-sweep/`); `migration/` gained one in this pass.

## PUBLIC IMPORT SURFACE

Consumers import the root barrel or a `package.json` subpath — `./atomic-write`, `./runtime`, `./xdg-data-dir`, `./logger`, `./record-type-guard`, `./archive-entry-validator`, `./classify-path-environment`, `./command-executor` (+`/*`), `./git-worktree` (+`/*`), `./internal-initiator-marker`, `./skill-path-resolver`, `./migration` (+`/*`), `./process-sweep`, `./process-tree`, `./prompt-async-gate`, and more — never a deep relative path. All subpaths point at source `.ts` (no dist). `package.json` `exports` is the authoritative list.

## WHERE TO LOOK (direct files beside their subdirs)

| File | Relationship |
|------|--------------|
| `migration.ts` | public `./migration` entry — re-exports the four map modules (agent/hook/model/category names); `migrateConfigFile()` itself lives in `migration/config-migration.ts` |
| `command-executor.ts` | flat sibling of `command-executor/` |
| `process-tree.ts`, `process-tree-termination.ts`, `process-stream-reader.ts` | flat process family (no subdir) |
| `prompt-async-gate.ts` | flat entry over `prompt-async-gate/` |
| `zip-entry-listing.ts` | flat entry over `zip-entry-listing/` |
| `skill-path-resolver.ts`, `internal-initiator-marker.ts` | adapter-facing helpers exported as subpaths |

## CONVENTIONS

- New single-function utility -> flat file + colocated test. New stateful/multi-module domain -> subdir + own barrel (+ subpath export if public).
- Only `src/index.ts` is package-public; subdirs keep their own `index.ts`.

## ANTI-PATTERNS

- Do not deep-import a subdir module when a subpath export exists.
- Do not move files between flat and subdir without updating the `exports` map.

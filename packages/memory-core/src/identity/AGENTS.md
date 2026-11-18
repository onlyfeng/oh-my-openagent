# identity - Memory Identity Resolution

**Score:** 12 (14 files, 2k LOC, index barrel, security-sensitive, confinement-tested). Parent [`packages/memory-core/AGENTS.md`](../../../AGENTS.md) documents `resolveAutoIdentity` and the opt-in `"git"` mode; this file maps the modules and their hard rules.

## OVERVIEW

Resolves WHICH memory repository a session uses: memory-root layout, project scope, linked-worktree mapping, and the stable identity slug.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Root layout - `MEMORY_ROOT_ENV_VAR`, `defaultMemoryRoot`, `resolveMemoryRoot`, `buildIdentityPaths`, directory names | `layout.ts` |
| Identity slug + resolvers - `MemoryIdentity` | `resolve.ts` |
| Project scope - `ProjectIdentity`, `createProjectIdentityResolver`, `PROJECT_SCOPE_SCAN_LIMIT` | `project-scope.ts` |
| Filesystem probe seam - `ProjectScopeFs`, `createProjectScopeProbe`, `settle` | `project-scope-fs.ts` |
| Linked-worktree → main-tree mapping - `GitEntryKind`, `GitLayoutProbe`, `resolveLinkedWorktreeSource` | `git-layout.ts` |
| Barrel (layout + project-scope + resolve) | `index.ts` |

## CONVENTIONS

- `project-scope-fs.ts` is the ONLY filesystem seam: production code reaches disk through it so tests inject probes instead of fixtures on real paths.
- `git-layout.ts` reads the `.git` pointer, `commondir`, the admin gitdir back-pointer and `core.worktree`/`core.bare` directly - it never spawns `git`.
- Scope probing is bounded by `PROJECT_SCOPE_SCAN_LIMIT` and cannot escape the session/project root.

## ANTI-PATTERNS

- Never let hostile paths or symlinks resolve outside the intended memory/project scope (`confinement.test.ts` pins this).
- Never return the session's own identity as a candidate during resolution.
- Never string-guess a git layout: unreadable, forged, or ambiguous layouts keep the path-derived id (`resolve-legacy.test.ts`, `resolve-worktree.test.ts`).
- Slugs are bounded and sanitized (`sanitize.test.ts`); do not widen the accepted character set casually.

## COMMANDS

```bash
bun test --timeout 20000 packages/memory-core/src/identity
bun run --cwd packages/memory-core typecheck
```

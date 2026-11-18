# src/tools - Memory Tool Operations

**Generated:** 2026-11-18 / c2691818a9

**Score:** 25 (30 files, ~4.7k LOC, star barrel; the `memory` + `memory_apply_patch` operation layer).

## OVERVIEW

Operations behind the memory tools: path-validated mutations over the git MemFS, Codex-style multi-file patches, provenance stamping, the shared locked commit helper, and typed tool errors. Barrel star-exports `memory`, `claim-frontmatter`, `commit-write`, `patch-apply`, `patch-parser`, `tool-errors`.

## WHERE TO LOOK

| Task | File |
|------|------|
| `memory` ops (`create`/`str_replace`/`insert`/`delete`/`rename`/`update_description`) | `memory.ts` |
| Multi-file Codex patches | `memory-apply-patch.ts` + `patch-parser.ts` + `patch-apply.ts` |
| Locked commit helper | `commit-write.ts` - `commitMemoryWrite`: memory-write lock, clean-tree check, effective-change check, agreements-violation check for memory-tool writes, commit, secret + stranded-lock failure handling |
| Provenance | `claim-frontmatter.ts` - `stampReceipt`, `restampContent`, legacy-header checks, trailer formatting, `readHeader` |
| Leaked-argument repair | `leaked-arguments.ts` - `repairLeakedArguments`, `describeRepairs` |
| Rollback | `rollback-behind-lock.ts` (`pending-rollback` tests pin it) |
| Guards + errors | `agreements-write-guard.ts` (automation cannot touch `by: human` items), `tool-errors.ts` |

## CONVENTIONS

- One operation per tool call, under the `memory-write` lock; commit only the affected paths.
- Provenance lives in frontmatter + commit trailers; every write carries it.
- Ambiguous leaked arguments are reported, never guessed into a repair.

## ANTI-PATTERNS

- Never bypass `commitMemoryWrite` on a mutating tool path.
- Never let `memory_apply_patch` skip validation to "recover" from a parse error.

Parent: [`../../AGENTS.md`](../../AGENTS.md) (one `src/tools/` row; this file is its detail).

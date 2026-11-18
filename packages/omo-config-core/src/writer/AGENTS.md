# src/writer/ — Comment-Preserving Config Writer

## OVERVIEW

7 files, ~870 LOC. Atomic, comment-preserving writes and surgical edits to
`omo.json[c]` — the write side paired with the loader. Barrel `index.ts`
exports `updateOmoConfig`, `applyOmoConfigEdit`, types, and
`DEFAULT_WRITE_FILE_SYSTEM`. Distinct domain: JSONC round-trip fidelity.
Parent: [`packages/omo-config-core/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `writer.ts` | `updateOmoConfig` — read-modify-write with atomic publication |
| `surgical-edit.ts` | `applyOmoConfigEdit` — targeted text edits preserving comments/formatting |
| `types.ts` | `OmoConfigEdit`, `OmoConfigWriteFileSystem`, options/results |
| `writer-security.test.ts` | Traversal/unsafe-write guards |
| `surgical-edit.test.ts` | Comment/format preservation coverage |

## CONVENTIONS

- Writes go through the injected `OmoConfigWriteFileSystem`
  (`DEFAULT_WRITE_FILE_SYSTEM` for production) — never direct fs in logic.
- Filesystem side effects of migrations belong to the migration engine, not
  here; this module only owns config-file text manipulation.
- `writer-security.test.ts` is a gate: keep path validation strict when
  touching writer internals.

## ANTI-PATTERNS

- Never write config non-atomically or without re-parsing the result.
- Do not break comment preservation for the sake of a simpler serializer —
  that is the module's reason to exist.

## QA

```bash
tsgo --noEmit -p packages/omo-config-core/tsconfig.json
bun test packages/omo-config-core
```

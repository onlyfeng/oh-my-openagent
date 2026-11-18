# src/memfs/ — Memory Markdown Substrate

## OVERVIEW

29 files, ~4.1k LOC. Memory-path validation, strict YAML frontmatter parsing,
expiry handling, and hook-script installation for the git-backed memory
repository. Public surface is the `index.ts` barrel re-exporting `hooks`,
`hooks-scripts`, `frontmatter`, `expiry`, `frontmatter-entries`,
`lookalike-text`, `frontmatter-validation`, `normalize`, and `paths`.
Parent: [`packages/memory-core/AGENTS.md`](../../AGENTS.md).

## WHERE TO LOOK

| Area | Role |
|------|------|
| `frontmatter.ts`, `frontmatter-scalar.ts`, `frontmatter-validation.ts` | Strict shared frontmatter grammar; `renderMemoryFile` quoting/round-trip rules |
| `frontmatter-entries.ts`, `normalize.ts` | Entry parsing and one-time legacy header normalization |
| `hooks.ts`, `hooks-scripts.ts` | Hook-script installation and provenance (`hooks-commit-msg.ts`) |
| `expiry.ts` | Entry expiry computation |
| `lookalike-text.ts` | Lookalike/normalization text helpers |
| `paths.ts` | Memory-path validation (repository-relative, normalized) |

## CONVENTIONS

- One frontmatter grammar everywhere: any scalar the `yaml` package would not
  read back verbatim is quoted; the writer re-parses its own header.
- Tests are colocated and heavily table-driven (`frontmatter-strict-yaml`,
  `hooks-secret`, `hooks-provenance`, `hooks-agreements-provenance`,
  `hooks-teardown` suites pin the grammar).
- Hook-script content and provenance properties are contract-tested; new hook
  scripts go through `hooks-scripts.ts`, never ad-hoc writes.

## ANTI-PATTERNS

- Never emit an unquoted `description` frontmatter scalar by hand.
- Never bypass the barrel when adding public surface — importers use
  `memory-core` package paths.
- Do not loosen the strict YAML grammar to accept legacy headers; legacy input
  flows through `normalize.ts` instead.

## QA

```bash
bun test packages/memory-core/src/memfs/
bun run --cwd packages/memory-core typecheck
```

# skills/programming/scripts/ — Language Scaffolding + Rule Checkers

## OVERVIEW

36 tracked files, ~4,030 LOC across TypeScript, Go, Python, shell, and
templates. Executable helpers backing the `programming` skill: per-language
new-project/new-script scaffolders, "no-excuse" rule checkers, and shared
tool-runner logic. **Score 15** (36 files, multi-language, own entry points).
Parent: [`skills/programming/SKILL.md`](../SKILL.md).

## WHERE TO LOOK

| Area | Role |
|------|------|
| `typescript/`, `python/`, `go/`, `rust/` | Per-language scaffolders (`new-project.ts`, `python/new-script.ts`) and rule checkers (`check-no-excuse-rules.ts` 407 LOC, `check-no-excuse-rules.py` 688 LOC) |
| `shared/run-tool.ts` | `runTool` / `assertNever` — shared runner invoking `uv`, `cargo`, or `go` for scaffold validation; distinguishes unavailable tools from failed runs |
| `*/templates/` | Scaffolding assets per language |
| `scaffolders.test.ts` | Cross-language scaffolder suite |

## CONVENTIONS

- Grouped by target language; shared execution logic stays in `shared/`.
- Rule checkers are runnable directly with their language runtime
  (`bun run` / `python` / `bash`).
- Scaffolds are validated by actually running the toolchain via
  `runTool` — not by static checks alone.

## ANTI-PATTERNS

- Do not duplicate runner logic per language; extend `shared/run-tool.ts`.
- Do not treat a missing toolchain (`uv`/`cargo`/`go`) as a scaffold
  failure — report it as unavailable.

## QA

```bash
bun test packages/shared-skills/skills/programming/scripts
```

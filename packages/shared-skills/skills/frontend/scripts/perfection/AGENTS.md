# perfection — Standalone Frontend Perfection Checkers

**Generated:** 2026-11-18 / c2691818a9

**Score:** 16 (62 files: 21 TS/Python sources, ~2.1k LOC, + 41 fixtures; 3 subdirs; 2 CLI entries). Sibling of [`detect/`](../detect/AGENTS.md): detect renders and lints pages; perfection lints source text and fixtures standalone. `collectFiles` here is also detect's file collector (`detect/cli.ts` imports it).

## OVERVIEW

Two standalone CLI checkers plus a Chrome audit script: accent-stripe linting and draft-guard (non-rendered text) linting, each with positive/negative fixtures.

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Accent stripes | `check-accent-stripes.ts` (CLI) + `accent-stripes/` (`markup.ts`, `declarations.ts`, `color.ts`, `css-parse.ts`, `gradient.ts`, `selector.ts`, `tailwind.ts`, `findings.ts`, `lint.ts`) |
| Draft guard | `check-draft-guard.ts` (CLI) + `draft-guard/` (incl. `text-nodes.ts`) |
| Shared collection | `collect-files.ts` — `collectFiles`, recursive file/dir targets |
| Browser audit | `lighthouse-audit.py` (real Chrome required) |
| Fixtures | `fixtures/{accent-stripes,draft-guard}/` — `positive`/`negative` expected cases per checker |

## CONVENTIONS

- Checkers accept file OR directory targets; findings report stable file/line/column.
- `--json` switches output; exit codes are a contract: 0 clean, 1 findings, 2 usage/unreadable target.
- `text-nodes.ts` blanks `NEVER_RENDERED` content before scanning — hidden/non-rendered text is not a visible-copy finding.

## ANTI-PATTERNS

- NEVER use the `lighthouse` CLI in `lighthouse-audit.py` — it drives headless-shell, not real Chrome (stated in the script header).
- Do not duplicate collection logic in a checker; everything goes through `collectFiles`.
- Do not treat fixture `negative` cases as findings — they assert the checker stays silent.

## COMMANDS

```bash
bun packages/shared-skills/skills/frontend/scripts/perfection/check-accent-stripes.ts [--json] <file-or-dir>...
bun packages/shared-skills/skills/frontend/scripts/perfection/check-draft-guard.ts [--json] <file-or-dir>...
bun test packages/shared-skills/skills/frontend/scripts/perfection/
```

# src/hooks/directory-readme-injector/ — README Context Injection

## OVERVIEW

8 files, ~520 LOC. Tool-guard hook that injects ancestor `README.md` context
on file reads: walks from the read file's directory toward the project root,
finds READMEs, injects them root-first through the shared truncator, and
caches injected directories per session. Wired by
`create-tool-guard-hooks.ts` behind `isHookEnabled("directory-readme-injector")`.
Parent: [`src/hooks/AGENTS.md`](../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `hook.ts` | `createDirectoryReadmeInjectorHook` — read-tool completion handling |
| `finder.ts` | Walk from file dir to project root locating `README.md` |
| `injector.ts` | Root-first injection through the shared dynamic truncator |
| `storage.ts` | Persistent injected-path state under OpenCode storage `directory-readme/` |
| `constants.ts` | Cache keys and limits |

## CONVENTIONS

- Per-session in-memory cache of injected directories; persistent state keyed
  under the shared OpenCode storage directory.
- Session state is cleared on deleted/compacted session events (handled in
  `hook.ts`).
- Truncation uses the shared truncator — no local cutoff logic.

## ANTI-PATTERNS

- Do not inject a directory's README twice in one session.
- Do not introduce a bespoke truncation path; reuse the shared one so budgets
  stay consistent with other injectors.

## QA

```bash
bun test packages/omo-opencode/src/hooks/directory-readme-injector
bun run --cwd packages/omo-opencode typecheck
```

Any change to this wired hook requires real OpenCode QA (root AGENTS.md).

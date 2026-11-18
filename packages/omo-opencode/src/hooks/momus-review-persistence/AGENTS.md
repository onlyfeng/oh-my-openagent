# src/hooks/momus-review-persistence/ — Momus Review Persistence

## OVERVIEW

4 files, ~590 LOC. Persists Momus task review results: parses verdict blocks,
resolves the single contained plan path, and records review metadata through
`tool.execute.before/after` handlers. Instantiated in
`create-tool-guard-hooks.ts` behind `isHookEnabled("momus-review-persistence")`.
Parent: [`src/hooks/AGENTS.md`](../AGENTS.md).

## WHERE TO LOOK

| File | Role |
|------|------|
| `index.ts` | Barrel exporting `createMomusReviewPersistenceHook` |
| `hook.ts` | Factory returning the before/after tool handlers |
| `review-resolution.ts` | `parseMomusVerdict`, `extractMomusVerdictBlock`, `extractSinglePlanPath`, `readReviewMetadata`, `resolveSessionReviewContext` |
| `hook.test.ts` | Colocated suite (temp project dirs, cleaned up) |

## CONVENTIONS

- Only Momus task results carrying a verdict AND exactly one contained plan
  path are persisted; ambiguity means skip, not guess.
- Missing prompt/model/duration can be recovered from child session messages
  (`resolveSessionReviewContext`).
- Paths outside the project plans tree are rejected.

## ANTI-PATTERNS

- Never persist a review with multiple or unresolved plan references.
- Never accept a plan path escaping the project plans directory.

## QA

```bash
bun test packages/omo-opencode/src/hooks/momus-review-persistence/hook.test.ts
bun run --cwd packages/omo-opencode typecheck
```

Changes to this wired hook require real OpenCode QA via the `opencode-qa`
skill with local `.omo/evidence/` artifacts.

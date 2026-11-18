# packages/omo-computer-engine

**Generated:** 2026-11-18 | c2691818a9

## OVERVIEW

Contract-only package. Today `src/index.ts` declares just `packageName` and `dependencyPackageNames` (the architecture edge to `@oh-my-opencode/omo-computer-protocol`). The package description promises locator/checksum/signature trust, stable-path launch, and bunshin descriptor — none of that exists in `src/` yet.

## STRUCTURE

```
omo-computer-engine/
├── package.json           # exports "." and "./types" straight to ./src/index.ts (TS source, not built)
├── tsconfig.json          # erasableSyntaxOnly: true; node types
├── vitest.config.ts       # dot reporter; github-actions reporter under CI
├── knip.workspace.jsonc   # includeEntryExports: true
├── src/index.ts           # packageName, dependencyPackageNames
└── test/smoke.test.ts     # vitest smoke
```

## ANTI-PATTERNS

- DO NOT treat this package as implemented. Adding real locator/launch behavior is new work, not a refactor.
- DO NOT normalize the test runner to bun test. `omo-computer-engine` is the one package that runs `vitest --run test` (own `vitest.config.ts`); migrate the package deliberately if you change it.
- Layered knip ownership: a new run-by-path file means registering it under `entry` in `knip.workspace.jsonc`.

## COMMANDS

```bash
bun run --cwd packages/omo-computer-engine test       # vitest --run test
bun run --cwd packages/omo-computer-engine typecheck  # tsgo --noEmit -p tsconfig.json
```

## CONSUMER

`packages/omo-computer-hand/src/index.ts` imports `packageName`. No other consumers.

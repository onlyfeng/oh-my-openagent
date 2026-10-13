# ulw-loop - goal-loop engine: plan state, checkpoint gates, spawn/stop hooks

**Score 17** (171 files recursive, ~18k LOC; own Biome/TS config, SDK + type barrels, highest reference centrality in the plugin tree).

## Stack

- Node >=20 runtime.
- npm package manager.
- TypeScript 7 strict mode.
- Biome 2 linting and formatting.
- Vitest 5 test runner.

## Forbidden

- No `as any` or `as unknown`.
- No `@ts-ignore` or `@ts-expect-error`.
- No enums.
- No non-null assertions.
- No default exports. `vitest.config.ts` is exempt because the framework requires that shape.

## File Ceiling

- Keep each `src/` TypeScript file under 250 pure LOC.
- Split by responsibility before a file reaches the ceiling.

## Test Discipline

- Use Vitest with nested `describe` names in `#given`, `#when`, and `#then` form, or inline `// given`, `// when`, and `// then` comments.
- Never use Arrange-Act-Assert comments.
- Keep fixtures in `test/fixtures/`.

## Commit Style

- Use Conventional Commits.
- Keep commits atomic.
- Each commit's tests and build must pass on its own.

## Branding

- Repo artifacts live under `.omo/ulw-loop/` paths.
- Environment variables use the `OMO_ULW_LOOP_*` prefix.
- CLI commands use the `omo-agent-toolkit ulw-loop` form in docs (aggregate plugin CLI namespace; this package's own bins are `omo-ulw-loop`, `ulw`, `ulw-loop`).
- Do not use any alternate legacy CLI alias anywhere.

## Layout

- `src/cli.ts`: bin entry (`omo-ulw-loop`, `ulw`, `ulw-loop` all map to `dist/cli.js`); documented invocation form `omo-agent-toolkit ulw-loop <subcommand>`.
- `src/cli-commands.ts`: subcommand dispatch (`ULW_LOOP_SUBCOMMANDS`, `ulwLoopCommand`, flag/value readers).
- `src/plan-io.ts`: plan persistence, append-only `ledger.jsonl`, `withUlwLoopMutationLock`.
- `src/quality-gate.ts` (221 LOC), `src/checkpoint.ts` (241), `src/steering.ts` (228): state-transition hotspots (evidence containment, checkpoint reconciliation, steering mutations); `src/state-lock.ts` (257) owns the persistence lock.
- `src/quality-gate-artifacts.ts`: artifact-ref / surface-kind primitives shared by the gate validator; `src/quality-gate-aggregate.ts`: non-throwing defect pre-pass that aggregates every gate defect into one error.
- `src/codex-hook.ts`: UserPromptSubmit steering injection + `create_goal` budget guard.
- `src/spawn-guard.ts` (288, largest src file), `src/spawn-role-guard.ts`, `src/spawn-budget-io.ts`, `src/stop-resume-hook.ts`: spawn admission/role/budget guards, Stop auto-resume. `src/sdk/` (`factory.ts`, `manifest.ts`, `types.ts`) wraps `plan-crud.ts`.
- `src/surface.ts`: toolkit surface + reviewer-identity resolution (`lazycodex` default; staged `surface.json` marker or `OMO_AGENT_TOOLKIT_SURFACE` selects `omo-senpi`).
- `src/ultrawork-skill-pointer.ts`: byte-identical mirror of ultrawork's pointer (pinned by `plugin/test/ultrawork-skill-pointer.test.mjs`).
- `directive.md`: runtime-read directive (never inlined into TypeScript). GENERATED from the canonical `packages/prompts-core/prompts/ultrawork/codex.md` by `components/ultrawork/scripts/sync-directive.mjs` and checked in, because this package is published standalone with no prompts-core dependency. Do not hand-edit; byte identity is pinned by `test/ultrawork-directive.test.ts`.

## Build and Hooks

- Build output goes to `dist/`.
- `hooks/hooks.json` wires `hook user-prompt-submit --with-ultrawork` (UserPromptSubmit), `hook pre-tool-use` (create_goal budget), `hook pre-tool-use-spawn` (spawn guards), `hook post-tool-use-spawn` (PostToolUse spawn admission recorder), and `hook stop` (auto-resume).

## Commands

- `npm test` (vitest --run) / `npm run test:watch`; focused: `bunx vitest run test/<file>.test.ts`
- `npm run typecheck` / `npm run lint` / `npm run build` (`tsc -p tsconfig.build.json` then `bun build src/cli.ts --target node --format esm`) / `npm run check`
- These stay npm even though the repo root is bun-driven: `.github/workflows/ci.yml` runs this component as `npm --prefix "$component" run check` (the loop selects every component whose `package.json` declares `scripts.check`, which includes this one), `scripts.check` itself shells `npm run build`, and the surrounding plugin tree is installed with `npm --prefix packages/omo-codex/plugin ci` in both `ci.yml` and `publish.yml`. Run the bun equivalents (`bun run test`, `bunx tsc -p tsconfig.build.json`, `bunx biome check .`) locally if you prefer, but do not rewrite this section to bun-only while CI invokes npm here.

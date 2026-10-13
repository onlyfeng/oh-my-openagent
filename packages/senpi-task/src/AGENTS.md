# senpi-task/src - Package Source Root

Source root of `@oh-my-opencode/senpi-task`: 19 direct files (8 implementation, 11 tests, ~3.6k LOC) plus 20 subsystem dirs. Earned its file because the direct layer owns the public barrel, the package subpath map, and the import guards every subsystem must respect. Subsystem maps live in the child AGENTS.md files linked below; the area table is in the package [`AGENTS.md`](../AGENTS.md).

## DIRECT FILES

| File | Role |
|------|------|
| `index.ts` (662) | Package-root barrel, re-exports ONLY (`allow: SIZE_OK`). Adding a symbol here makes it public to omo-senpi. |
| `model-chain.ts` | `buildRuntimeModelChain` / `chainRungCandidates` / `availableChainCandidates`: selected rung first, then the remaining chain candidates for runtime fallback. |
| `run-stats.ts` | `createRunStatsTracker`, `tokensPerSecond`: generation windows from the managed child event stream. |
| `progress.ts`, `status-line.ts`, `task-summary.ts` | Child progress rows, bounded status labels, `TASK_SUMMARY_MAX_LENGTH` force-truncation (harness side of the schema `maxLength`). |
| `notice-box.ts`, `renderer-text.ts` | Local copy of senpi `buildNoticeBox` (drop once the pinned senpi exports it); renderer width/excerpt helpers. |
| `senpi-static-import-guard.test.ts` | Fails if a module statically imports the senpi engine or pi-tui barrels instead of `lazy/`. |
| `senpi-api-tripwire.test.ts` | Pins the senpi API shapes this package depends on. |

## PACKAGE SUBPATHS (package.json `exports`)

`.` -> `index.ts`; `./agents-builtin` -> `agents/builtin/index.ts`; `./category-builtins` / `./category-resolver` / `./category-coverage` -> `category/{builtins,resolver,coverage}.ts`; `./dag` -> `dag/index.ts`; `./rpc-spawn` / `./rpc-model-admission` -> `runners/rpc/{spawn,model-admission}.ts`; `./task-renderers` -> `tools/task/renderers.ts`; `./notice-box`, `./renderer-text`. Moving any of these files is a cross-package break (omo-senpi, omo-native).

## SMALL SUBSYSTEMS (no own AGENTS.md)

| Dir | Key facts |
|-----|-----------|
| `kernel-tools/` | Parent kernel-tool grants. `resolveKernelToolGrant` (14 call sites) is all-or-nothing and resolved BEFORE child creation; capability is duck-typed (`contract.ts`) for pinned-engine skew; invoke scope is sent only when `capabilities.invokeScope === true`. Bindings are runtime-only, keyed by child task / pool id. |
| `eval-handles/` | `createEvalHandleHost` (`watch`/`result`/`send`/`cancel`/`output`); `watch.ts` subscribes before the initial snapshot, emits only increasing revisions, never polls. Completion refs are rejected. |
| `lazy/` | `loadSenpiBarrel` (20 files) / `loadPiTui`: memoized dynamic imports, state shared across bundled copies via `Symbol.for` keys; sync accessors throw while cold. |
| `isolation/` | Injectable port over `isolation-core` (runtime, prepare, settle, salvage, baseline-store, details). |
| `senpi/` | `explicit-pin.ts`, `minimal-resource-loader.ts` (a supplied system prompt REPLACES senpi's dynamic prompt), `thinking-level.ts` (`none` -> `off`, `auto` -> undefined, unknown tokens dropped). |
| `config/` | Only `settings.test.ts`; task settings resolve in `@oh-my-opencode/omo-config-core`, never here. |
| `__adversarial__/` | Seeded chaos harness (`prng.ts`, `chaos-harness.ts`, `chaos-invariants.ts`, host-chaos variants); every draw derives from the captured seed. |

## WHERE TO LOOK

| Task | Location |
|------|----------|
| Task records, locks, state dir | [`store/AGENTS.md`](store/AGENTS.md) |
| Spawn, concurrency, respawn, workpool launch | [`manager/AGENTS.md`](manager/AGENTS.md) |
| Suspend, reconcile, revival, TTL, destroy | [`lifecycle/AGENTS.md`](lifecycle/AGENTS.md) |
| Child execution (in-process, process, host session) | [`runners/AGENTS.md`](runners/AGENTS.md), [`runners/rpc-host/AGENTS.md`](runners/rpc-host/AGENTS.md) |
| DAG engine | [`dag/AGENTS.md`](dag/AGENTS.md) |
| `task` tool module map | [`tools/task/AGENTS.md`](tools/task/AGENTS.md) |

## ANTI-PATTERNS

- NEVER value-import `@code-yeongyu/senpi` or `@earendil-works/pi-tui` statically under `src/` (type-only imports are allowed); go through `lazy/`, or every rpc child pays the barrel at boot (guard test fails).
- NEVER put logic in `index.ts`; it stays a re-export barrel.
- NEVER give curated read-only agents parent kernel tools, read-only ones included (`kernel-tools/resolve.ts`).
- Without invoke-scope capability, refuse a narrowed grant instead of exposing write-capable parent tools.
- NEVER invoke a revived child's parent kernel tool through an unavailable-binding stub; missing bindings fail closed (`kernel-tools/wrapper.ts`).
- NEVER auto-merge crash salvage (`isolation/salvage.ts`); a failed clone refuses the spawn, never runs in the parent checkout (`isolation/prepare.ts`).
- A task pin may not accept and silently drop a service tier; unrecognized `:` suffixes stay on the model id (`senpi/explicit-pin.ts`).

## COMMANDS

```sh
bun test packages/senpi-task/src/*.test.ts        # direct-layer guards and render helpers
bun test packages/senpi-task/src/__adversarial__  # chaos bench; SEED=<label> reruns a seed
bun packages/senpi-task/scripts/manual-qa.ts <evidence-dir>   # store + transitions; also manual-{agents,category,output}-qa.ts
bun packages/senpi-task/scripts/manual-routing-policy-qa.ts    # prints routing-policy scenarios
```

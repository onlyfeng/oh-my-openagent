# tests/ - Repository-Level Integration Tests

## OVERVIEW

Cross-package invariants and integration fixtures that do not belong to one package's co-located unit suite.

## STRUCTURE

```
tests/
├── omo-config-category-drift.test.ts  # Config/category contract stays synchronized
├── omo-schema-freshness.test.ts       # Generated omo.schema.json matches source
├── omo-schema-unique-ids.test.ts      # omo.schema.json gives each $id one schema; profiles $ref the [opencode] block (issue #6444)
├── schema-defaulted-fields-optional.test.ts  # committed config schemas never list a defaulted field as required (issue #6445)
├── reasoning-vocabulary-parity.test.ts  # model-core vs omo-config-core reasoning levels + normalization agree
├── ulw-loop-define-goal-reference.test.ts  # ulw-loop skills (omo-senpi + omo-codex editions) ship define-goal.md beside full-workflow.md
├── ulw-plan-review-convergence-contract.test.ts  # bounded review-convergence contract JSON present in every edition's full-workflow.md (issues #6128, #10378)
└── hashline/                          # Standalone headless Hashline exercise package
    ├── package.json / bun.lock / tsconfig.json
    ├── knip.workspace.jsonc  # Per-directory knip fragment composed by script/knip-config.ts
    ├── headless.ts           # Live-model entry; imports adapter-internal hashline modules
    ├── test-environment.ts   # Required env parser (HTTP/HTTPS URL, non-blank key)
    └── test-*.ts             # Plain scripts spawning headless.ts (not bun:test)
```

## TEST SURFACES

| Surface | Command | Purpose |
|---------|---------|---------|
| Root invariants | `bun test tests/*.test.ts` | Fast cross-package contract checks (direct children only; the glob deliberately misses `hashline/`) |
| Full root suite | `bun test` | Includes these tests through root `bunfig.toml` |
| Hashline fixture | Run from a disposable full-repository copy inside a filesystem-isolated VM/container | Exercises edit operations and multi-model behavior through adapter-internal imports; makes live network/model calls, and may write model-selected paths relative to cwd |

## CONVENTIONS

- Keep package-specific tests beside package source; use this directory only for real cross-package or standalone-fixture boundaries.
- **Prompt/prose contract tests are forbidden.** Do not pin authored prompt, skill, rule, `AGENTS.md`, or markdown-instruction wording or structure. Allowed seams are machine-consumed fields/sentinels/tool names, byte or shipped-copy equality between real artifacts, and observable runtime behavior such as parsing, routing, dispatch, state, security, and dynamic input propagation.
- Treat `tests/hashline/` as its own Bun package. Preserve its lockfile and headless entry rather than importing it into the root test preload. Its heavy drivers (`test-edit-ops.ts`, `test-edge-cases.ts`, `test-multi-model.ts`) are plain Bun scripts that spawn `bun run headless.ts -p <prompt>` subprocesses against per-case temp demo files and validate the edited content - not `bun:test` cases - so they never match Bun's `*.test.ts` discovery and the root suite only ever picks up the self-contained `test-environment.test.ts` parser test.
- `tests/hashline/knip.workspace.jsonc` is this directory's knip fragment, composed into the root config by `script/knip-config.ts` (#13200). Every new runnable file must be added to its `entry` list; `script/knip-entry-exports.test.ts` fails on a missing entry. `headless.ts` and the `ai` / `@ai-sdk/openai-compatible` dependencies are deliberate pre-existing ignores - do not delete them to silence knip.
- The Hashline fixture imports adapter-internal `../../packages/...` paths, performs live network/model calls, and may write model-selected paths relative to cwd. Run it only from a disposable full-repository copy inside a filesystem-isolated VM or container with no sensitive parent files and never from the primary checkout or an active worktree.
- In that sandbox, set explicit short-lived, least-privilege test-only values for `HASHLINE_TEST_BASE_URL` and `HASHLINE_TEST_API_KEY`. `test-environment.ts` requires and validates them (HTTP/HTTPS URL, non-blank key); `headless.ts` throws without them and no longer falls back to hardcoded defaults. Use a non-production endpoint and never use production or personal credentials.
- Keep generated files and raw stdout/stderr inside the disposable sandbox or local redacted evidence directory. Redact before sharing and never publish raw logs.
- Freshness tests compare generated artifacts to source-derived output; regenerate the artifact instead of changing the expected value manually.
- Pure prose changes do not get phrase-pin tests. Test machine-consumed schemas, registration, or runtime behavior.

## ANTI-PATTERNS

- Do not turn `tests/` into a second home for ordinary package unit tests.
- Do not remove, skip, or isolate a failing cross-package invariant to make the root suite green.
- The synthetic AGENTS.md benchmark fixture under `packages/omo-opencode/src/__tests__/perf/fixtures/` is governed by its own local guide; this guide only cross-references it and does not govern its contents.

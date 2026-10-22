# omo-senpi src

Source root of the Senpi adapter package. The package barrel (`index.ts`) exports only `omoSenpiAdapterPackageName` and re-exports `./install`; everything else is consumed through `extension/index.ts` (extension entry) and per-component barrels. Package anatomy, build, and QA live one level up in `../AGENTS.md`.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Extension entry / composition | `extension/` | `index.ts` (source entry, eager task), `bundled-index.ts` (built entry, lazy task runtime). Own AGENTS.md. |
| Install / uninstall | `install/` | `runSenpiInstaller` / `runSenpiUninstaller`, local launcher, atomic settings writes. |
| Real host modules for tests | `senpi-test-runtime.ts` | Resolves the installed `@code-yeongyu/senpi` dist and imports real theme/ModelRegistry/ModelRuntime at load time. |
| Deep components | `components/{memory,task,lsp,telemetry,init-deep-advisor,thread,gateway}/` | Each has its own AGENTS.md; `memory/` additionally documents `worker/`, `commands/`, `palace/`, `kibitzer/`. |
| Bundled remote MCPs | `components/builtin-mcps/` | `context7` + `grep_app` HTTP declarations, `CONTEXT7_API_KEY` bearer gating, and the disable switches. Own AGENTS.md. |
| X search | `components/x-search/` | Credential-gated `x_search` tool and conditional skill (files, gating, contract, error codes, backtest pointer). Own AGENTS.md. |
| Small components | `components/*` | Single-purpose factories (ulw-loop, side-panel, onboarding, config-watch, fallback-architect, ...) documented in `../AGENTS.md`. |

`src/components/` holds 35 component dirs; 9 of them carry their own AGENTS.md (the deep rows above plus `builtin-mcps` and `x-search`). A new component registers in `extension/component-list.ts` and gets documented in `../AGENTS.md` unless it earns a deeper file.

## Root audit gates

Colocated at `src/` root: 17 executable package-contract tests, not documentation:

- `bundle-purity.test.ts` - import allowlist for the extension bundle; keep aligned with `SENPI_LOADER_ALIASES` in `plugin/scripts/build-extension.mjs` (peer-external rule in `../AGENTS.md`).
- `bundle-size.test.ts` - raw byte budget for built `plugin/extensions/omo.js`. Every ceiling increase requires a separate measured PR with bundle sizes before/after and the reason; never fold it into a feature PR.
- `bundle-lazy-barrel.test.ts` - lazy dynamic imports survive minification in the emitted bundle.
- `bundled-skill-disable.test.ts` - a `disabled_skills` entry drops that skill from the resolved native skill set.
- `computer-use-lazy-runtime.test.ts` - asserted on the EMITTED artifacts: computer-use keeps only its registration shell in `omo.js`; the desktop client and handle ship in `omo-computer-use.js` and load on first use.
- `hermetic-types.test.ts` - type surface stays hermetic for the packaged build.
- `posthog-sidecar.test.ts`, `posthog-packed-install.test.ts` - the telemetry transport sidecar resolves and ships correctly in a packed install.
- `package-shape.test.ts` - adapter manifest contract; license/notice files must ship with generated artifacts.
- `plugin-manifest.test.ts` - packaged plugin manifest.
- `runtime-dependency-resolution.test.ts` - a symlinked plugin without host hoisting still resolves runtime deps from the real path.
- `runtime-package.test.ts` - LSP daemon runtime staging: manifest pins sorted outputs; tampered output rejected.
- `extension-node-runtime-audit.test.ts` - extension loads under plain Node/jiti: no Bun-only module properties (`import.meta.dir` / `import.meta.file`) at module scope (v5.0.0-beta.1 regression).
- `senpi-main-runtime-import-audit.test.ts`, `omo-native-capture-path.audit.test.ts` - import/capture-path surface audits.
- `skills-sync.test.ts` - synced skills carry no foreign harness tokens (`codex`, `multi_agent`, `spawn_agent`, case-insensitive).
- `windows-console-hide.test.ts` - every production `node:child_process` call in the package passes `windowsHide: true`, so no spawn opens a console window that steals the user's focus on win32. It resolves the entry points each file imports (the whole `spawn`/`exec`/`fork` family, aliases included) and walks the source tree, because the hand-listed `spawn(`-only predecessor missed the `execFile` probes behind #8501. A foreground process that must keep the user's console opts out with a `windowsHide-exempt:` comment stating why, at the call site.

## CONVENTIONS

- Tests colocated (`*.test.ts`), Bun runner, `#given ... #when ... #then` names; `FakeExtensionAPI` drives registration without a host.
- Components are factory objects (`create*Component()`) with `register(pi, ctx)`; registration order lives in `extension/component-list.ts`.
- Host surface is consumed only through `extension/types.ts` structural ports (`SenpiExtensionAPI`, `ComponentContext`), never concrete senpi types outside `extension/`.

## COMMANDS

```bash
bun test packages/omo-senpi                                   # package suite
tsgo --noEmit -p packages/omo-senpi/tsconfig.json             # typecheck
bun run test:senpi                                            # full gate: build + stage + typecheck + tests
```

## ANTI-PATTERNS

- Don't loosen an audit gate to unblock a build; fix the violation or deliberately change the contract.
- Don't add runtime dependencies without keeping `runtime-dependency-resolution.test.ts` / `runtime-package.test.ts` green from a symlinked install.
- Don't grow the package barrel; new surface goes through `extension/` or a component barrel.

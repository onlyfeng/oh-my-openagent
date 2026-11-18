# bin/ - Published Launcher Shim

## OVERVIEW

Node-side launch surface for every public CLI alias. The root shim resolves a platform package and spawns its generated `bin/oh-my-opencode.js` launcher payload; it is not the TypeScript CLI implementation.

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Main launcher | `oh-my-opencode.js` | Entry for `oh-my-opencode`, `oh-my-openagent`, `omo-agent-toolkit`, `lazycodex`, `lazycodex-ai` (root `package.json` `bin`) |
| Platform selection | `platform.js` | OS/arch/libc package candidates and baseline fallback; imported by `postinstall.mjs` |
| Platform types | `platform.d.ts` | Hand-written declarations for the plain-JS selector; nothing imports it (knip-ignored) |
| Version mismatch output | `version-mismatch.js` | Wrapper/platform-package mismatch diagnostics (issue #3918); imported by `postinstall.mjs` |
| Knip workspace config | `knip.workspace.jsonc` | Directory-owned entry composed by root `script/knip-config.ts` |
| Behavior tests | `*.test.ts` | Bun tests: invocation routing, candidate ordering, installer delegation, mismatch semantics |

## RUNTIME FLOW

`oh-my-opencode.js` derives the invocation name (`OMO_INVOCATION_NAME` env passthrough, else `argv[1]` basename), routes `lazycodex`/`lazycodex-ai` `update`/`uninstall` to `packages/omo-codex/scripts/install-local.mjs` before platform resolution, resolves ordered `oh-my-opencode-*`/`oh-my-openagent-*` candidates, and spawns the first installed one with `OMO_INVOCATION_NAME` and `OMO_WRAPPER_PACKAGE_ROOT` set. On x64 it probes AVX2 (`/proc/cpuinfo`, macOS `sysctl`); `OH_MY_OPENCODE_FORCE_BASELINE=1` forces baseline-first ordering. A spawn error or `SIGILL` with a remaining candidate falls through to the next; other signals exit 128+N (SIGINT 130, SIGILL 132, SIGKILL 137, SIGTERM 143). Generated platform payloads run root `dist/cli` with Bun, falling back to `dist/cli-node` with Node, so a Bun `SIGILL` is normally handled inside the selected launcher rather than by switching baseline packages.

## CONVENTIONS

- Plain Node only: the shim and generated payloads must work from the published package layout.
- Preserve all five aliases. Scoped wrapper names reduce to the bare name first; `lazycodex`/`lazycodex-ai` intentionally resolve to the `oh-my-openagent` platform family (`PLATFORM_PACKAGE_BASE_BY_WRAPPER_NAME`).
- `win32` maps to `windows`; musl appends `-musl`; unknown Linux libc must throw, never resolve silently.
- Version comparison strips only a leading `v`; prerelease suffixes stay significant (`4.5.1-beta.1` != `4.5.1`).
- Keep error output actionable: detected platform, ordered candidates, and the `npm install <first candidate>` hint.
- Tests run from the root with `bun test bin` or as part of `bun test`.

## ANTI-PATTERNS

- Do not import adapter source from `packages/omo-opencode/src/`; this shim must remain usable from the published package layout.
- Do not hard-code one npm package family. `oh-my-opencode` and `oh-my-openagent` wrappers share the launcher.
- Do not edit generated launcher payloads under `packages/oh-my-opencode-*/bin/`; regenerate them through `script/build-binaries.ts`.
- Do not rename/remove exports of `platform.js`/`version-mismatch.js` without updating `postinstall.mjs`.
- Do not remove baseline fallback without updating platform tests and publish-platform coverage.

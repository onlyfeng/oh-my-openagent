# senpi-desktop-prelude - `computer` Prelude Assets (Desktop)

**Generated:** 2026-11-18 / c2691818a9 | Earned by score 13 (31 files, generated-asset pipeline, hard package boundary). One export: `computerPreludeAssets` (`src/index.ts`) - the eval-kernel facades and model documentation for the `computer` tool.

## WHAT SHIPS

| Key | Source | Use |
|-----|--------|-----|
| `javascript` / `python` | `src/prelude.js` / `src/prelude.py` | Define `globalThis.computer` (JS) / `computer` (Python, synchronous) in the eval kernels |
| `declarations` | `declarations.d.ts` | TS declarations of the JS `computer` global |
| `documentation` / `safety` | `docs/computer.md`, `docs/computer-safety.md` | `<prelude>` block helper list (no code fences - it renders inside one); system-prompt safety fragment |
| `exports` | - | `["computer"]`, the kernel globals the snippets define |
| `methodAllowlist` | - | Every call-chain method: union of the `desktop-protocol` tier tables |

`javascript` + `python` + `documentation` + `exports` form the tool's `kernelPrelude`.

## GENERATION

`facade-manifest.json` (709 lines) is the source of truth; `scripts/generate-assets.ts` compiles it into `src/assets.generated.ts` / `.json` (lazy getters keep the published object shape without carrying asset bytes per session). Generated regions in `src/index.ts` are marked - run `bun run generate` after manifest edits.

## TOOL CONTRACT (the facades rely on it)

Every helper is one ordinary `tool.computer(args)` call - activation and the permission preflight apply to it. `call` chains carry at most two `{ method, args }` steps (desktop root, then window/element); handles re-resolve on every call. `run` serializes `fn` + args into `return await (fn)({ desktop, wait, assert }, ...args)`. Facades read `{ text, details?, images?, hasError? }`: `hasError` throws, images go to `display()`, `details.value` returns.

## BOUNDARY

The desktop-package boundary test (`scripts/desktop-package-boundaries.test.mjs`, run from `senpi-desktop-service`) enforces: this package imports NO workspace package; only the desktop `tool` and `service` import the desktop packages; no desktop package imports `pi-agent-core`, `pi-ai`, or `pi-tui`.

## ANTI-PATTERNS

- NEVER hand-edit `src/assets.generated.*` or the generated regions - change `facade-manifest.json` and regenerate.
- Do not bypass the `computer` tool contract with direct facade transport.
- Do not add workspace imports (the boundary test fails).

## COMMANDS

```bash
bun run generate   # after manifest edits
bun run typecheck
bun run test       # vitest --run test; pins drift, byte identity, facade methods, allowlist
```

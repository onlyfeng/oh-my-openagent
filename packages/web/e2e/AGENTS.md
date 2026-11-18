# e2e/ - Playwright Browser Suite

**Generated:** 2026-11-18 / c2691818a9 | **Score:** 10 (23 files, ~2.7k LOC - the site's visual + behavioral contract)

## OVERVIEW

18 Playwright specs plus shared helpers and a local release-fixture server. Two-config split: the default `../playwright.config.ts` IGNORES `download-flag-off.spec.ts`, while `../playwright.flag-off.config.ts` runs ONLY that spec against its own port-3200 build with `OMO_WEB_DESKTOP_DOWNLOAD` unset. `bun run test:e2e` (parent SCRIPTS) runs flag-off first, then the main suite; root `bun test` never sees this directory (root `bunfig.toml` excludes `packages/web`).

## WHERE TO LOOK

| File(s)                                                                                                                                                  | Role                                                                                                                                                                                                  |
| -------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `download.spec.ts` (395 LOC)                                                                                                                             | Download-page flows. `test.describe.configure({ mode: "serial" })` - scenarios mutate the ONE shared fixture host; each test pins browser `userAgentData` so OS/arch asset selection is deterministic |
| `download-flag-off.spec.ts`                                                                                                                              | Gate audit: with the launch flag off, no entry point may leak or 404 the download page                                                                                                                |
| `manifesto*.spec.ts` (5) + `manifesto-page.ts` + `manifesto-glyphs.ts`                                                                                   | Manifesto page behavior; helpers `waitForReadingBlocks` / `firstProgress` and the `expectUniformWordGlyphs` glyph oracle                                                                              |
| `secret-reading-state.ts`                                                                                                                                | `scrollSecret` - reading-reveal state driving the lit-depth/reading specs                                                                                                                             |
| `support/download-fixture-server.ts`                                                                                                                     | Stands in for `get.omo.dev`: serves each channel's `manifest.json` + per-release `SHA256SUMS`; tests switch scenarios with `PUT /__scenario/<name>`                                                   |
| `support/download-fixture-port.ts`                                                                                                                       | `FIXTURE_PORT = 4310`                                                                                                                                                                                 |
| `landing`, `hero-*`, `responsive`, `focus-ring`, `tooltip-sheet`, `docs*`, `document-shell`, `work-surfaces`, `support.spec.ts`, `dag-dimmed-legibility` | Page-level visual/behavior specs                                                                                                                                                                      |

## CONVENTIONS

- Specs run headless with `colorScheme: "dark"` (written against the Night palette; a Day check sets the scheme itself) and `--use-mock-keychain` so a dev machine's real keychain/profile is never touched.
- Assertions target stable `data-testid` selectors and observed behavior, with `// given` / `// when` / `// then` comments (never AAA).
- Never hit the real `get.omo.dev` or any live host - route release traffic through `support/download-fixture-server.ts`.

## COMMANDS

```bash
bun run test:e2e                        # from packages/web: flag-off config, then default
bunx playwright test download.spec.ts   # focused (default config)
```

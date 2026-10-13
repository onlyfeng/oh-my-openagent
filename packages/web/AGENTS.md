# packages/web/ — Marketing Site (Next.js + Cloudflare Workers)

**Generated:** 2026-10-13 (dc373d26f; prior f3642fcda, 2026-05-20)

## OVERVIEW

Public-facing marketing site for OmO (omo.dev). Next.js 16 (App Router) deployed to Cloudflare Workers via [@opennextjs/cloudflare](https://opennext.js.org/cloudflare). Independent of the npm plugin — its own `package.json`, `bun.lock`, and `tsconfig.json`.

## STACK

| Layer          | Choice                                                                                                                                                        |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Framework      | Next.js 16.3 (App Router, RSC, Turbopack)                                                                                                                     |
| Runtime target | Cloudflare Workers (`compatibility_flags: ["nodejs_compat"]`)                                                                                                 |
| Adapter        | `@opennextjs/cloudflare` (build → `.open-next/worker.js`)                                                                                                     |
| Styling        | Tailwind v4 (`@tailwindcss/postcss`) + shadcn/ui (`components.json`)                                                                                          |
| i18n           | `next-intl` with `app/[locale]/...` routing; 4 locales (en/ja/ko/zh) in `messages/`                                                                           |
| Animation      | CSS keyframes + Tailwind `animate-*`; **no motion library dep** (never framer-motion, per DESIGN.md). 3D landing graph uses `three` + `@react-three/{fiber,drei}` (`components/landing/graph/`) |
| E2E / unit     | Playwright (`e2e/*.spec.ts`, chromium + firefox in CI); `bun test lib` for colocated `lib/*.test.{ts,mjs}`                                                    |
| Lint/Format    | Biome 2.5 lint + ESLint 10 Next flat config + Prettier (Tailwind plugin)                                                                                      |

## STRUCTURE

```
packages/web/
├── app/[locale]/         # localized routes (App Router)
├── components/           # by domain: landing/ (dag/, graph/, crafted/, sections/), manifesto/, docs/, ledger/ + ui/ (shadcn), icons/; no barrels
├── lib/                  # cn, stats (shared cached aggregation), install-targets, docs-source/sections/page, download-badge, og/, asset-retention
├── messages/{en,ja,ko,zh}.json  # i18n strings
├── i18n/                 # next-intl request/routing config
├── proxy.ts              # next-intl routing and canonical redirects
├── public/               # static assets (largest dir, ~4 MB)
├── e2e/                  # Playwright tests
├── scripts/              # prepare-build (purge .next/cache/fetch-cache + regen docs), generate-docs-content, generate-og-fonts, render-app-icons, render-graph-poster, check-graph-budget, check-deploy-coherence, retain-static-assets
├── next.config.ts
├── open-next.config.ts
├── wrangler.toml         # worker name + compatibility settings
├── playwright.config.ts
├── eslint.config.mjs
├── postcss.config.mjs
├── tsconfig.json
├── components.json       # shadcn config
└── package.json
```

## SCRIPTS

```bash
# from packages/web/ directory
bun install && bun run dev   # next dev (local Node.js)
bun run lint             # biome lint + eslint (lint:fix to write)
bun run format           # prettier --write
bun run format:check
bun run type-check       # generate-docs-content.mjs, then tsgo --noEmit
bun run build            # prebuild regenerates docs; bun --bun next build
bun run preview          # opennextjs-cloudflare build + preview locally
bun run deploy           # opennextjs-cloudflare build + deploy to Cloudflare
bun run test             # bun test lib
bun run test:e2e         # playwright test
bun run icons:check      # committed PNG icons match a fresh render of app/icon.svg (icons:render rewrites)
bun run cf-typegen       # regenerate cloudflare-env.d.ts from wrangler.toml bindings
```

## CI/CD

| Workflow                           | Trigger                                                          | What                                                                    |
| ---------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `.github/workflows/web-ci.yml`     | push/PR to master/dev touching `packages/web/**`, `docs/**`, or the workflow | format check, lint, type-check, unit tests, next build, opennextjs-cloudflare build, Playwright e2e |
| `.github/workflows/web-deploy.yml` | push to master/dev touching `packages/web/**`, `docs/**`, or the workflow, OR manual dispatch | full deploy via `cloudflare/wrangler-action@v4`                         |

**Account and secret:** the Worker runs on the Sisyphus Labs Cloudflare account (moved 2026-10-09); `wrangler.toml` pins the account id and names the omo.dev zone routes by `zone_id`. The deploy workflow uses the `omo-production` GitHub environment, whose `CLOUDFLARE_API_TOKEN` can edit Workers scripts and the omo.dev zone's routes and nothing else (sisyphuslabs/common-infra `docs/cutover-workers.md`).

## RELATIONSHIP TO npm PACKAGE

The root npm package ships `dist/`, `bin/`, `postinstall.mjs` plus selected sibling artifacts (see root `package.json` `files`). `packages/web/` is **never** included in any npm publish; it is exclusively a separate Cloudflare deployment target.

Root `bun test` ignores `packages/web/**` through `bunfig.toml` so `packages/web/e2e/*.spec.ts` does not pollute plugin tests.

## CONVENTIONS

- **No path aliases globally** in the omo project, but `packages/web/` is a Next.js app where `@/*` aliases are the framework default. Keep `@/*` confined to packages/web/.
- Use the existing shadcn primitives in `components/ui/` rather than installing new UI libs.
- All user-facing copy goes through `messages/{locale}.json`; never hardcode strings in components.
- **`DESIGN.md` is the design source of truth** — tokens, near-black `#0a0a0a`/`#08090a` (never pure `#000`/`#fff`), Geist typography, `min-h-[100dvh]` over `h-screen`, IntersectionObserver over scroll listeners, only `transform`/`opacity` animated.
- Generated files carry a `DO NOT EDIT` marker: `lib/docs-content.generated.ts` (from root `docs/` via `scripts/generate-docs-content.mjs`) and `lib/og/fonts.ts` (`scripts/generate-og-fonts.mjs`). Edit the source and regenerate.
- Format with prettier before commit; `web-ci.yml` enforces `format:check`.

## ANTI-PATTERNS

- Never run `npm install` in `packages/web/`. Use `bun install` only. (Root `.gitignore` already blocks `package-lock.json`.)
- Never commit `.next/`, `.open-next/`, `.wrangler/`, `node_modules/` (covered by `packages/web/.gitignore`).
- Never deploy locally with `bun run deploy` against production — use the GitHub Actions workflow so Cloudflare credentials live in one place.
- Do not add `framer-motion`; CSS keyframes/Tailwind animation only (narrow `motion/react layoutId` exception in DESIGN.md). No serif typography; no layout-property animation (`width`/`height`/`top`/`left`/margin/padding).
- Known accepted debt: docs pages overflow horizontally at 390×844 — do not worsen without the docs-shell design work.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# packages/web/ — Marketing Site (Next.js + Cloudflare Workers)

**Generated:** 2026-11-18 / c2691818a9 (update pass; prior 2026-08-24 f3642fcda)

## OVERVIEW

Public-facing marketing site for OmO (omo.dev). Next.js 16 (App Router) deployed to Cloudflare Workers via [@opennextjs/cloudflare](https://opennext.js.org/cloudflare). Independent of the npm plugin — its own `package.json`, `bun.lock`, and `tsconfig.json`. NOT part of the npm package (root `package.json` `files` ships only `dist/`, `bin/`, `postinstall.mjs`) — exclusively a Cloudflare deployment target; root `bun test` ignores `packages/web/**` via `bunfig.toml`.

## STACK

| Layer          | Choice                                                                                                                                                        |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Framework      | Next.js 16.3 (App Router, RSC, Turbopack)                                                                                                                     |
| Runtime target | Cloudflare Workers (`compatibility_flags: ["nodejs_compat"]`)                                                                                                 |
| Adapter        | `@opennextjs/cloudflare` (build → `.open-next/worker.js`)                                                                                                     |
| Styling        | Tailwind v4 (`@tailwindcss/postcss`) + shadcn/ui (`components.json`)                                                                                          |
| i18n           | `next-intl` with `app/[locale]/...` routing; 4 locales (en/ja/ko/zh) in `messages/`                                                                           |
| Animation      | CSS keyframes + Tailwind `animate-*`; **no motion library dep** (`motion/react` only for shared layout transitions — never full framer-motion, per DESIGN.md) |
| E2E            | Playwright (`e2e/*.spec.ts`)                                                                                                                                  |
| Lint/Format    | ESLint 10 + Next flat config + Prettier (Tailwind plugin)                                                                                                     |

## STRUCTURE

```
packages/web/
├── app/[locale]/               # localized routes; tokens in app/styles/design-system.css
├── components/                 # UI primitives, shadcn components/ui/, home/, docs/
├── lib/ i18n/ proxy.ts         # helpers (cn, build-id), next-intl config, canonical redirects
├── messages/{en,ja,ko,zh}.json # all user-facing copy; public/ static assets; e2e/ Playwright specs
└── scripts/                    # prepare-build.mjs, generate-docs-content.mjs, icon/poster utils
```

## SCRIPTS

```bash
# from packages/web/ (bun install ONLY — never npm install)
bun run dev              # next dev (local Node.js)
bun run lint | lint:fix  # eslint (cached; app/components/lib/e2e/scripts)
bun run format | format:check    # prettier --write / --check
bun run type-check       # generate-docs-content + tsgo --noEmit
bun run build            # bun --bun next build (Turbopack)
bun run test             # bun test lib
bun run test:e2e         # playwright: flag-off config first, then the default config
bun run preview | deploy # opennextjs-cloudflare build + preview locally / deploy to Cloudflare
bun run cf-typegen       # regenerate cloudflare-env.d.ts from wrangler.toml bindings
```
Asset utilities: `icons:render` / `icons:check` (app icons), `poster:graph` / `check:graph-budget`.

## CI/CD

| Workflow                           | Trigger                                                          | What                                                                    |
| ---------------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------------------- |
| `.github/workflows/web-ci.yml`     | push/PR to master/dev that touches `packages/web/**`             | format check, lint, type-check, next build, opennextjs-cloudflare build |
| `.github/workflows/web-deploy.yml` | push to master that touches `packages/web/**` OR manual dispatch | full deploy via `cloudflare/wrangler-action@v3`                         |

**Account/secret:** the Worker runs on the Sisyphus Labs Cloudflare account (moved 2026-10-09); `wrangler.toml` pins the account id and names the omo.dev zone routes by `zone_id`; the `omo-production` GitHub environment's `CLOUDFLARE_API_TOKEN` can edit Workers scripts and the omo.dev zone's routes and nothing else (sisyphuslabs/common-infra `docs/cutover-workers.md`). **Build id** = commit sha (`lib/build-id.mjs`), so `/_next/static/<sha>/` belongs to one commit; re-deploying the same sha with non-byte-identical rebuilt manifests is stopped by the asset-retention guard (`Content-addressed asset changed bytes`) — the guard failing safe, not a bug to bypass: deploy a new commit instead.

## CONVENTIONS

- `@/*` aliases are the Next.js framework default HERE only (the omo project has no global path aliases); reuse the existing shadcn primitives in `components/ui/` rather than installing new UI libs; all user-facing copy goes through `messages/{locale}.json`, never hardcoded.
- **Design source of truth:** the tokens in `app/styles/design-system.css` (doriterm.com's Night and Day palettes, light and dark, Timeless Sans + JetBrains Mono) and the landing system in `components/home/styles`. `DESIGN.md` still describes the earlier dark-only Geist design and is being rewritten (#12658); where they differ, the stylesheet wins. IntersectionObserver over scroll listeners; only `transform`/`opacity` animated.
- Docs content under `components/docs/` is generated by `scripts/generate-docs-content.mjs` (DO NOT EDIT marker) — edit the source docs, not the generated output. Format with prettier before commit — `web-ci.yml` enforces `format:check`.

## ANTI-PATTERNS

- Never run `npm install` in `packages/web/` — `bun install` only (root `.gitignore` already blocks `package-lock.json`). Never commit `.next/`, `.open-next/`, `.wrangler/`, `node_modules/` (covered by `packages/web/.gitignore`). Never deploy locally with `bun run deploy` against production — use the GitHub Actions workflow so Cloudflare credentials live in one place.
- Do not add `framer-motion`; CSS keyframes/Tailwind animation only (narrow `motion/react layoutId` exception in DESIGN.md). No serif typography; no layout-property animation (`width`/`height`/`top`/`left`/margin/padding).
- Known accepted debt: docs pages overflow horizontally at 390×844 — do not worsen without the docs-shell design work.

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

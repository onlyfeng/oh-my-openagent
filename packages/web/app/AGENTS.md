# web/app/ - Next.js App Router Surface

**Score ~14** (41 files; Next App Router routes/layouts/metadata, distinct from `web/components`).

## OVERVIEW

The Next.js App Router tree for the `web` package: localized route groups, route-local metadata, API `route.ts` handlers, and the design-token stylesheet. UI building blocks live one level over in [`../components`](../components/AGENTS.md); this directory is routing, metadata, and app shell only.

## WHERE TO LOOK

| Area                                       | Role                                                                                               |
| ------------------------------------------ | -------------------------------------------------------------------------------------------------- |
| `layout.tsx` / `[locale]/layout.tsx`       | App + localized home layout (`revalidate`, `viewport`, `generateMetadata`, `generateStaticParams`) |
| `[locale]/.../page.tsx`                    | Localized site/docs/download/manifesto/etc. routes                                                 |
| `app/api/*/route.ts`                       | API route handlers                                                                                 |
| `manifest.ts` / `robots.ts` / `sitemap.ts` | Static metadata generators                                                                         |
| `_components/`                             | Shared route-level components                                                                      |
| `styles/design-system.css`                 | Design tokens (~1.6k lines - the design-system concentration)                                      |

## CONVENTIONS

- Locale route groups with per-route `generateStaticParams` and route-local metadata.
- The design/showcase route reads `OMO_WEB_SHOWCASE=1` per request; `robots.ts` keeps showcase mode non-indexable.
- `@/*` aliases stay local to `packages/web`. Reuse `components/ui` shadcn primitives; route copy through `messages/{locale}.json`.

## ANTI-PATTERNS

- `robots.ts` must keep `OMO_WEB_SHOWCASE=1` non-indexable.
- Do not use `npm install` here - Bun only. Do not commit `.next/`, `.open-next/`, `.wrangler/`.
- Do not run production `bun run deploy` locally; deploy through GitHub Actions.

## COMMANDS

- From `packages/web/`: `bun run dev` · `bun run build` · `bun run type-check` · `bun run lint`

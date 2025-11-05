## Quick orientation

This is an Astro-based personal blog/portfolio scaffold wired to Cloudflare Workers. Key pieces:

- Framework: `astro` (see `astro.config.mjs`).
- Hosting/adapter: Cloudflare Workers via `@astrojs/cloudflare` and `wrangler` (see `wrangler.json` and `package.json`).
- Content: Markdown / MDX collections under `src/content/blog/` managed with Astro Content Collections (`src/content.config.ts`).
- Pages & routing: `src/pages/` maps to site routes. The blog index is `src/pages/blog/index.astro`; post routes use `src/pages/blog/[...slug].astro`.
- Layouts & components: `src/layouts/BlogPost.astro` and `src/components/*` (e.g. `BaseHead.astro`, `Header.astro`, `Footer.astro`) are the main UI building blocks.

## What to know (big picture)

- Content collection schema: `src/content.config.ts` defines a `blog` collection with a Zod schema. Frontmatter keys enforced: `title`, `description`, `pubDate` (coerced to Date), optional `updatedDate`, optional `heroImage`.
- The site pulls posts with `getCollection('blog')` (see `src/pages/blog/index.astro`) and sorts by `data.pubDate`.
- Templates render `post.data.*` directly (e.g. `heroImage` used as an `<img src=.../>` in `src/layouts/BlogPost.astro`). Images in `public/` are served from the web root.
- Build & deploy: `npm run build` creates `./dist/`, which is then deployed with `wrangler deploy` (see `package.json` scripts and `wrangler.json` mapping `assets.directory` to `./dist`).

## Developer workflows (commands)

Run these from the repository root.

```bash
npm install
npm run dev        # local dev server (astro) - default port 4321
npm run build      # astro build -> outputs to ./dist/
npm run preview    # builds and runs a local preview (uses wrangler dev)
npm run deploy     # build + wrangler deploy to Cloudflare
npm run check      # build + tsc + wrangler deploy --dry-run (CI-friendly smoke check)
npm run types      # generate wrangler types
```

Notes:
- `wrangler` is pinned in `devDependencies` (see `package.json`). Deployment requires Cloudflare credentials or CI secrets; `npm run check` is useful to run a dry-run before an actual deploy.
- `npm run preview` uses `wrangler dev` which proxies the built site locally; use when you want to validate Cloudflare-specific behavior.

## Project-specific conventions & patterns

- Content files must live in `src/content/blog/` and be `.md` or `.mdx` (the collection loader is configured with a glob in `src/content.config.ts`).
- Frontmatter must match the Zod schema; `pubDate` can be written as an ISO string and will be coerced to a Date.
- Routes: `src/pages/blog/index.astro` renders a list via `getCollection('blog')`. Single-post routes are handled by `src/pages/blog/[...slug].astro` (supporting nested slugs).
- Layouts expect `Astro.props` shaped as `CollectionEntry<'blog'>['data']` (see `src/layouts/BlogPost.astro`). When creating components or helper functions, follow this typing pattern.
- Static assets (fonts, images) go in `public/`. Example: hero images referenced in frontmatter are placed in `public/` and used directly as `post.data.heroImage`.

## Integration points & external dependencies

- Cloudflare Workers (via `@astrojs/cloudflare`) — `wrangler.json` indicates worker main script and assets binding. Observability and source map upload are enabled in `wrangler.json`.
- Astro integrations: `@astrojs/mdx`, `@astrojs/sitemap`, `@astrojs/rss` are used. MDX permits React/JSX inside posts (`using-mdx.mdx` example exists).
- TypeScript: project extends `astro/tsconfigs/strict` with `strictNullChecks`. Run `tsc` as part of CI or `npm run check`.

## Concrete examples (copy-paste patterns)

- New blog post frontmatter (place file under `src/content/blog/`):

```md
---
title: "My post"
description: "Short description"
pubDate: 2025-11-01
heroImage: "/blog-placeholder-1.jpg"
---

Post content here.
```

- Accessing posts in a page (see `src/pages/blog/index.astro`):

```ts
const posts = (await getCollection('blog')).sort((a,b) => b.data.pubDate.valueOf() - a.data.pubDate.valueOf());
```

## Files to inspect first (high-value)

- `astro.config.mjs` — integrations & adapter
- `package.json` — scripts and dependencies
- `wrangler.json` — Cloudflare worker & assets mapping
- `src/content.config.ts` — collection loader & schema
- `src/pages/blog/index.astro` and `src/pages/blog/[...slug].astro` — routing and rendering patterns
- `src/layouts/BlogPost.astro` — how post props are consumed
- `src/components/*` — shared UI pieces (meta, header, footer)

## Behavior to avoid or be careful with

- Do not assume `heroImage` is always present — `BlogPost.astro` checks for it before rendering.
- The content schema enforces dates; avoid introducing frontmatter keys outside the schema without updating `src/content.config.ts`.

## Quick verification steps after a change

1. Run `npm run dev` and open the dev server (default 4321).
2. For changes that affect build/deploy, run `npm run build` and `npm run preview`.
3. Run `npm run check` to run the build + typecheck + wrangler dry-run.

---

If any of these sections are unclear or you'd like examples for a specific task (adding a component, changing schema, or troubleshooting a deploy failure), tell me which area and I will expand the instructions.

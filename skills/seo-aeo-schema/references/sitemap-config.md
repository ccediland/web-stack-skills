---
title: "@astrojs/sitemap configuration"
summary: Pinned version, the opinionated config block, the on-demand-route inclusion pattern, lastmod wiring, output location, and known bugs.
last_updated: 2026-06-17
applies_to: "@astrojs/sitemap@3.7.3, astro@6.4.7, @astrojs/cloudflare@13.7.0"
---

# @astrojs/sitemap configuration

## Pin

Pin `@astrojs/sitemap@3.7.3`. Version 3.7.1 shipped a build-fail regression (a `reduce` on undefined); 3.7.3 fixes it. The integration is maintained by the Astro core team and tracks Astro 6.

```bash
npm i @astrojs/sitemap@3.7.3
```

## The opinionated config

The `site` URL is set in the root config, not inside `sitemap()`. The integration discovers build-time routes and writes `sitemap-index.xml` plus one or more `sitemap-0.xml` files.

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://example.com',
  integrations: [
    sitemap({
      namespaces: { news: false, video: false }, // a brand site rarely needs these
      filter: (page) => !page.includes('/draft/'),
      serialize(item) {
        // set item.lastmod here from your data source; see below
        return item;
      },
      entryLimit: 45000,
    }),
  ],
});
```

`changefreq` and `priority` are ignored by Google, so do not invest in them; `lastmod` is the field that matters. `namespaces` (added in 3.6.0) prunes unused XML namespaces (news, xhtml, image, video) to keep the file lean. `entryLimit` defaults to 45000 and triggers a sitemap index past that. `customPages` adds external or otherwise-undiscovered URLs as a static array.

## The on-demand-route limitation

The integration cannot analyze a page's source and cannot generate entries for on-demand routes. The Cloudflare adapter defaults to on-demand rendering, so only prerendered routes appear in the generated sitemap. `customPages` expects a static array and rejects an async function.

Two ways to include the rest:

- Keep content routes prerendered (`export const prerender = true` on the route or page), so the integration discovers them.
- Emit a custom prerendered sitemap endpoint that builds entries from content collections, for full control. This replaces or supplements the integration's output.

```ts
// src/pages/sitemap-custom.xml.ts
import type { APIRoute } from 'astro';
import { getCollection } from 'astro:content';
export const prerender = true;

export const GET: APIRoute = async ({ site }) => {
  const posts = await getCollection('blog');
  const urls = posts.map((p) => {
    const loc = new URL(`blog/${p.id}/`, site).href;
    const lastmod = p.data.updated ?? p.data.date;
    return `<url><loc>${loc}</loc><lastmod>${new Date(lastmod).toISOString()}</lastmod></url>`;
  }).join('');
  const xml = `<?xml version="1.0" encoding="UTF-8"?><urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">${urls}</urlset>`;
  return new Response(xml, { headers: { 'Content-Type': 'application/xml; charset=utf-8' } });
};
```

## lastmod wiring

Source `lastmod` from content-collection frontmatter or git modification time, and set it in `serialize`:

```js
serialize(item) {
  const date = lookupDateFor(item.url); // your map from URL to a real date
  if (date) item.lastmod = new Date(date).toISOString();
  return item;
}
```

Returning `undefined` from `serialize` drops the entry, which is another way to filter.

## Output location and known bugs

Under the Cloudflare adapter the static assets directory is `dist/client`, so the sitemap files land there and are served as static assets. Confirm a `.assetsignore` or routing rule does not exclude them.

Known bug 16838 (reproduces through 3.7.2): `lastmod` is not written into the `<sitemap>` entries of `sitemap-index.xml` even when `serialize` sets a per-URL `lastmod`. Per-URL `lastmod` inside each `sitemap-0.xml` is unaffected. A custom endpoint sidesteps this.

Reference the sitemap from robots.txt with a `Sitemap:` line (see `robots-and-llms.md`).

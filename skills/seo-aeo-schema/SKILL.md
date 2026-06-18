---
name: seo-aeo-schema
description: Add the machine-readable SEO and AEO layer to an Astro 6 site on Cloudflare Workers Static Assets — typed JSON-LD structured data, a sitemap, robots.txt, llms.txt, and a canonical and Open Graph head component. Use when adding schema.org structured data or a typed JSON-LD graph with schema-dts, wiring an entity graph that cross-links Organization, WebSite, WebPage, BreadcrumbList and a page entity, configuring @astrojs/sitemap, writing a robots.txt with AI-crawler directives, shipping an llms.txt, adding canonical or Open Graph or Twitter meta tags, or validating rich results. Explains why inline JSON-LD is exempt from Astro's hash-based CSP, why the sitemap integration misses on-demand routes, why llms.txt is an agent-readiness file rather than an SEO ranking lever, and which schema types still earn rich results in 2026. Trigger on JSON-LD, schema markup, structured data, sitemap, robots.txt, llms.txt, canonical tags, Open Graph, or rich results.
---

# SEO, AEO, and Structured Data for Astro 6 on Cloudflare

> The machine-readable discoverability layer for an Astro 6 site on Cloudflare Workers Static Assets: a typed JSON-LD @graph with schema-dts, a sitemap, robots.txt, llms.txt, and a canonical and Open Graph head component. Structured data earns rich results and entity recognition; it does not buy inclusion in AI answers.

## TL;DR

- Emit one centralized JSON-LD @graph per page with schema-dts 2.0.0, cross-linked by @id. schema-dts type-checks each node's properties but does NOT validate @id cross-references, so wire them by hand and validate the rendered output.
- Inline `<script type="application/ld+json">` is a non-executing data block, so CSP `script-src` does not apply and Astro 6's hash-based CSP needs no hash for it. Structured data is safe under the strict CSP from the web-security-headers skill. Still escape the characters that close a script tag.
- `@astrojs/sitemap` 3.7.3 only sees prerendered routes, because the Cloudflare adapter defaults to on-demand rendering. Include on-demand routes via a custom prerendered sitemap endpoint or `customPages`, and set `lastmod` through `serialize`.
- Ship robots.txt as the enforceable crawler-control surface, and llms.txt as an agent-readiness file. No major AI provider consumes llms.txt for ranking, so it is positioning, not SEO.
- FAQ and HowTo rich results are gone; FAQ was deprecated 7 May 2026. Author schema for what the page is, matched to visible content. Google requires no special schema for AI Overviews.

## Reference materials — load when relevant

This SKILL.md is the verdict, the layer split, and the minimal recipe. Load a reference only when its depth is needed:

- `references/json-ld-graph.md` — load when authoring the JSON-LD @graph: schema-dts 2.0.0 types, the canonical cross-@id entity graph, the render component, the @id non-validation limitation, and the CSP-exemption detail.
- `references/meta-and-canonical.md` — load when building the head component: canonical URL derivation, Open Graph, Twitter cards, the dynamic OG-image seam, and the hreflang seam.
- `references/sitemap-config.md` — load when configuring @astrojs/sitemap: the opinionated config, the on-demand-route inclusion pattern, lastmod wiring, output location, and known bugs.
- `references/robots-and-llms.md` — load when writing robots.txt or llms.txt: verified AI-crawler tokens, the access-versus-appearance rule, the llms.txt framing, and the shared prerendered text-endpoint pattern.
- `references/validation-and-aeo.md` — load when validating structured data or reasoning about AEO: validation tooling, the CI handoff, the 2026 rich-result types, and what JSON-LD does and does not do for AI surfaces.

## The layer

| Piece | What it does | Where it lives |
|---|---|---|
| JSON-LD @graph | Entity graph for rich results and entity recognition | One render component in the base layout |
| Sitemap | Crawl map for search engines | `@astrojs/sitemap`, prerendered routes |
| robots.txt | Crawler access control for search and AI | Prerendered endpoint or `public/` |
| llms.txt | Agent-readiness index, B2A not SEO | Prerendered endpoint or `public/` |
| Head and meta | Canonical, Open Graph, Twitter | Typed head component in the layout |

## Pinned versions

Verified against npm on 2026-06-17. Re-verify before adopting.

| Package | Version | Note |
|---|---|---|
| schema-dts | 2.0.0 | devDependency, TypeScript-only, zero runtime |
| @astrojs/sitemap | 3.7.3 | the 3.7.1 build regression is fixed |
| astro | 6.4.7 | baseline, requires Node 22 |
| @astrojs/cloudflare | 13.7.0 | emits no static headers, so CSP ships via a meta element |

## Setup in order

1. Install: `npm i -D schema-dts` and `npm i @astrojs/sitemap@3.7.3`. Add `sitemap()` to `astro.config.mjs` and set the `site` URL in the root config.
2. Build the typed @graph render component and include it in the base layout. See `references/json-ld-graph.md`.
3. Add the head component (canonical, Open Graph, Twitter) to the base layout. See `references/meta-and-canonical.md`.
4. Add `src/pages/robots.txt.ts` and `src/pages/llms.txt.ts` as prerendered endpoints. See `references/robots-and-llms.md`.
5. Validate the rendered structured data, then wire the exported CI script into the perf-ci-gates workflow. See `references/validation-and-aeo.md`.

## Gotchas

- schema-dts does not validate @id links. A cross-reference that points at a missing or wrong-typed node type-checks fine and silently breaks the graph, so validate the rendered JSON-LD, not just the types.
- The sitemap integration cannot see on-demand routes and cannot read a page's source. `customPages` rejects an async function. Keep content routes prerendered or emit a custom sitemap endpoint.
- `changefreq` and `priority` are ignored by Google; only `lastmod` matters. Through 3.7.2, bug 16838 drops `lastmod` from `sitemap-index.xml` entries even when `serialize` sets it.
- On the Cloudflare adapter every text endpoint needs `export const prerender = true`, or it renders on demand in the Worker instead of becoming a static file in `dist/client`.
- Disallowing an AI crawler in robots.txt does not remove a page from AI answers; that needs `noindex` or auth. Keep Googlebot allowed or you lose Search and AI Overviews.

## Limitations

- This skill does NOT cover: the CI workflow itself, which the perf-ci-gates skill owns and which this skill feeds with an exported validation script; RSS feeds, which are a content concern; full internationalization and hreflang, covered as a seam only; dynamic Open Graph image generation, covered as a seam only; and per-vertical schema design beyond the baseline graph.
- JSON-LD does NOT cause inclusion in AI Overviews, AI Mode, or chatbot citations. Its value is rich-result eligibility plus entity clarity, and the markup must match visible page content.
- llms.txt is an agent-readiness and Lighthouse 13.3 signal, not a search-ranking factor. Ship it as positioning, not for SEO.
- Pins verified 2026-06-17. Re-verify schema-dts and @astrojs/sitemap before adopting, and note that the Rich Results Test loses FAQ support in June 2026.

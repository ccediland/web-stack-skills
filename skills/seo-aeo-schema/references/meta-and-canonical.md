---
title: Canonical and Open Graph head component
summary: A hand-rolled typed head component for canonical URLs, Open Graph, and Twitter cards, plus the dynamic OG-image seam and the hreflang seam.
last_updated: 2026-06-17
applies_to: astro@6.4.7, @astrojs/cloudflare@13.7.0
---

# Canonical and Open Graph head component

## Why hand-rolled

A dedicated SEO integration adds a dependency and a layer for what is a dozen lines of head markup. Native-first wins here: a small typed Astro component owns title, description, canonical, Open Graph, and Twitter, with no third-party package. The component reads `Astro.site` and `Astro.url`, so canonical derivation is deterministic and needs no per-page wiring.

## Canonical URL derivation

Derive the canonical from `Astro.site` plus the current path, not from a hand-typed string, so it cannot drift. Match the site's trailing-slash convention, because Astro treats `/page` and `/page/` as distinct URLs and a mismatched canonical confuses crawlers.

```ts
const canonical = new URL(Astro.url.pathname, Astro.site).href;
```

`Astro.site` is the `site` value from the root config, which the sitemap also requires. If `Astro.site` is undefined the build has no `site` set; fix the config rather than falling back to a relative URL.

## The head component

```astro
---
interface Props {
  title: string;
  description: string;
  ogType?: 'website' | 'article';
  image?: string;
}
const { title, description, ogType = 'website', image } = Astro.props;
const canonical = new URL(Astro.url.pathname, Astro.site).href;
const ogImage = image ? new URL(image, Astro.site).href : undefined;
---
<title>{title}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonical} />

<meta property="og:type" content={ogType} />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:url" content={canonical} />
{ogImage && <meta property="og:image" content={ogImage} />}

<meta name="twitter:card" content={ogImage ? 'summary_large_image' : 'summary'} />
<meta name="twitter:title" content={title} />
<meta name="twitter:description" content={description} />
{ogImage && <meta name="twitter:image" content={ogImage} />}
```

Include this in the base layout's head, alongside the JSON-LD render component. Keep title and description consistent with the values fed into the JSON-LD graph and with visible page content; mismatched metadata is a quality signal against the page.

## The dynamic OG-image seam

This component consumes an OG image by URL; it does not generate one. A static per-page image referenced through the `image` prop is the simplest path and covers most premium sites.

Dynamic generation (rendering a per-page image from a template with satori or a similar library) is a real option but a separate concern, and on Cloudflare it carries constraints: the renderer runs in the Worker, fonts must be loaded as buffers, and the SVG-to-raster step needs a WASM build of resvg. Treat dynamic OG images as a seam to wire deliberately, not part of this baseline. The static-image path through this component is the default.

## The hreflang seam

Single-locale sites need nothing here. For a multi-locale site, two pieces cooperate: `@astrojs/sitemap` has an `i18n` option that emits `xhtml:link` alternates in the sitemap (see `sitemap-config.md`), and each page adds `<link rel="alternate" hreflang="...">` tags in the head for its locale variants. This skill covers the seam only; a full internationalization system is out of scope.

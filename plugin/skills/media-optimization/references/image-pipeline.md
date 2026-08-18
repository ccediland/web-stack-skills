---
title: The build-time image pipeline — service, responsive config, LCP, formats, SVG
summary: The exact adapter and image config, the Image/Picture usage rules, priority semantics (Astro sets nothing automatically), remote-image authorization, format and quality guidance, and the SVG posture.
last_updated: 2026-08-17
applies_to: astro@7.2.2 (astro:assets, responsive images stable since 5.10) · @astrojs/cloudflare@14.2.1
---

# The build-time image pipeline

> One config block makes every image responsive and build-time-optimized; after it, authoring is `<Image>` with an alt and — on exactly one image per page — `priority`.

## Contents

- The config block
- Image and Picture usage
- The LCP image and priority
- Remote images
- Formats and quality
- SVG posture
- Gotchas

## The config block

```js
// astro.config.mjs
import cloudflare from '@astrojs/cloudflare';

export default defineConfig({
  adapter: cloudflare({
    imageService: 'compile', // NEVER omit: the adapter default is the metered
    // cloudflare-binding service (default flipped in v13.0.0) whose free tier
    // fails closed at 5k unique transformations/month (error 9422).
  }),
  image: {
    layout: 'constrained',    // global default: auto srcset + sizes per image
    responsiveStyles: true,   // emits the zero-specificity :where() base styles
  },
});
```

- `compile` = sharp transforms at build time for prerendered routes; on-demand routes get passthrough (fine — on-demand on this stack is endpoints, not image pages). Equivalent compound form: `imageService: { build: 'compile' }`.
- `layout` values per image: `constrained` (scales down, never up — the default for content images), `full-width` (hero/banner strips), `fixed` (icons, avatars — density srcset only), `none` (opt out entirely).
- `responsiveStyles` emits a handful of global `:where([data-astro-image])` rules; `:where()` keeps them at zero specificity so any project CSS overrides them. Astro's CSP feature hashes the styles it emits — verify the pages stay clean under the site's CSP in preview, as with any new global style source.

## Image and Picture usage

```astro
---
import { Image } from 'astro:assets';
import banner from '../assets/banner.jpg';
---
<Image src={banner} alt="" layout="full-width" />
```

- Local images live in `src/` (usually `src/assets/`) and are IMPORTED — that is what puts them through the pipeline. Files in `public/` are served verbatim, untouched, unhashed: `public/` is for exact-file assets (brand logos, favicons — brand-canon-ingest territory), never for photos.
- With a global `layout` set, srcset and sizes are generated from the image's dimensions and layout type; hand-written `sizes` disappears from authoring. Override `widths`/`sizes` only for genuine art direction, via `<Picture>` with `pictureAttributes` on those slots alone.
- `alt` is required by the component; empty `alt=""` is a deliberate decorative-image statement, not a default.

## The LCP image and priority

```astro
<Image src={hero} alt="…" priority />
```

- `priority` sets exactly `loading="eager" decoding="sync" fetchpriority="high"`. Astro sets NONE of that automatically — every image defaults to lazy; an unmarked LCP image is the single most common self-inflicted LCP regression on image-led pages.
- Exactly ONE priority image per page — the LCP candidate above the fold. Two priorities compete for bandwidth and both lose.
- Pages whose LCP is text or a canvas (the WebGL hero pattern) have NO priority image; do not decorate.

## Remote images

Remote URLs are optimized only when authorized: `image.domains: ['cdn.example.com']` or `image.remotePatterns`. Unauthorized remote images pass through untransformed (they still get CLS-preventing dimensions when inferable). On this stack remote imagery usually signals a CMS or DAM decision that belongs upstream — prefer committing originals to the repo where the build can reach them.

## Formats and quality

- Default output format is webp; it is the right single-format answer for content images. `<Picture formats={['avif', 'webp']}>` buys AVIF's extra ~20-30% on hero-weight images at the cost of double transforms per image — spend it on the heaviest few, not globally.
- `quality` presets (`low`/`mid`/`high`/`max` or 0-100): the default is right until a specific image proves otherwise; fix outliers per image, not globally.
- Per-format encoder options live under `image.service.config` (jpeg/webp/avif/png objects) for when a project needs global encoder tuning — rare.

## SVG posture

- Importing `.svg` as a component (stable since 5.7) inlines it into the HTML — right for icons and small graphics; attributes (`width`, `fill`, …) pass through.
- `experimental.svgOptimizer` (svgo-based) is still experimental in 7.2.2 — do not enable in client work; run svgo manually in the asset workflow if SVG weight matters.
- `image.dangerouslyProcessSVG` stays FALSE (its default): rasterizing SVGs through the pipeline is a DoS surface and almost never what a brief means.
- Brand SVGs (logos) are exact-file artifacts under provenance — they bypass this pipeline entirely and are inlined verbatim where scheme-reactive (the display-none-twins lesson from the composition playbook).

## Gotchas

- The transform pipeline runs at build time — image-heavy sites pay it in build minutes. Thousands of originals × multiple widths is where `compile` build times get real; that is m1's documented flip, not a reason to hand-roll.
- `getImage()` exists for programmatic transforms (OG-image-adjacent work); OG rasters for the brand pipeline stay with brand-canon-ingest's recipe.
- Astro emits no automatic `<link rel="preload">` for images; `priority`'s fetchpriority covers the LCP case — do not add manual preloads on top (they double-fetch when srcset picks a different candidate).

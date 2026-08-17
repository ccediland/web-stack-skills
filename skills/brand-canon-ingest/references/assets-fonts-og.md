---
title: Assets, fonts, favicon, and OG image
summary: Exact-file logo and currentColor icon rules, the @font-face recipe with token-matched family names, favicon derivation from the mono isotype, and the raster OG recipe under the canon's image rules.
last_updated: 2026-08-17
applies_to: "brand-system-skills 0.6.0 · astro@7.2.2"
---

# Assets, fonts, favicon, and OG image

Load when wiring any visual asset from the brand repo into the site.

## Logos — exact-file, always

The canon's logo discipline (G-LOGO-01) resolves arrangement → treatment → EXACT FILE: the site ships the shipped SVG bytes, never a redraw, recolor, or distortion. Practical consequences:

- Copy the chosen variants from `assets/logos/` into the site (`src/assets/` for `astro:assets`-processed use, `public/` for verbatim URLs). Renaming to the site's file convention is fine — re-saving through an SVG optimizer is not (byte changes break the custody chain and can alter geometry).
- Which variant to use where is a canon call, not a taste call: the visual keystone and the grammar govern arrangement (horizontal, vertical, claim, isotype, wordmark) and treatment per background. On dark grounds the scheme declares the single luminous ink.
- The `-fc` full-color variants are OFF-SYSTEM (G-LOGO-02 — never FC): they exist in the repo as history, and they never ship on the site.
- The logo's own inks are explicit fills inside each file — logos do NOT re-ink with the scheme. Picking a different ink means picking a different file.

## Icons — currentColor, scheme-reactive for free

The icon set is single-stroke `currentColor`. Inline the SVG (Astro component or fragment import) so `currentColor` resolves from the surrounding text color — icons then follow every scheme switch with zero extra wiring. Don't hardcode fills into icon files, and don't load them as `<img>` when the color must track the scheme (external image documents can't inherit `currentColor`).

## Fonts — only the woff2 the repo ships

The brand repo ships TTF masters plus the web set: woff2 for the weights the brand actually uses. Declare `@font-face` ONLY for shipped woff2 — never fake missing weights (the browser would synthesize them off-brand):

```css
/* src/styles/fonts.css — Furever example: display face ships one weight, body face ships the range */
@font-face {
  font-family: 'Behind The Nineties';
  src: url('/fonts/behindthenineties-extrabold.woff2') format('woff2');
  font-weight: 800;
  font-style: normal;
  font-display: swap;
}
@font-face {
  font-family: 'Onest';
  src: url('/fonts/onest-regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
/* repeat per shipped Onest weight the site actually uses (500/600/700...) */

html { font-synthesis-weight: none; }   /* no faux-bold where a weight is missing */
```

Rules that make this work:

- The `font-family` name must byte-match the first entry of the token stack (`base.font-family.*` → `"Behind The Nineties"`, `"Onest"`); a mismatch silently falls back to `system-ui` everywhere.
- The fallback stack comes from the tokens — components use `var(--ds-base-font-family-display)` (or the semantic alias), never a hand-typed stack.
- Preload the display face used above the fold: `<link rel="preload" href="/fonts/behindthenineties-extrabold.woff2" as="font" type="font/woff2" crossorigin />`.
- Licensing: webfont files are owner-supplied; their licenses live with the owner, not the repo. Confirm with the owner that public self-hosting is licensed BEFORE go-live (a published open license like SIL OFL covers it; a purchased display face may not). Record the confirmation in the site's decision log.

## Favicon — derive from the mono isotype

The favicon derives from the shipped mono isotype (`<brand>-iso-mono-<ink>.svg`), never from a redraw. Default ink = the first single-ink treatment the grammar orders (for Furever, POL); the owner may override. Pasteable recipe:

```bash
cp <brand-repo>/assets/logos/furever-iso-mono-pol.svg public/favicon.svg
npx --yes sharp-cli -i public/favicon.svg -o public/apple-touch-icon.png resize 180 180
```

```html
<link rel="icon" href="/favicon.svg" type="image/svg+xml" />
<link rel="apple-touch-icon" href="/apple-touch-icon.png" />
```

The SVG favicon is the primary (crisp at every size); the 180px PNG covers Apple touch. Check both renders visually — tiny sizes can need the simpler isotype variant.

## OG image — raster, on-system, real logo

The canon's image rule (G-IMG-03) binds the OG/social preview: it must be RASTER (SVG previews break on major platforms), with no off-system color, no AI-generated animal/person, and no FC logo. Recipe — author a 1200×630 HTML template that uses the ingested tokens and a real logo file, then screenshot it:

```bash
npx --yes playwright install chromium          # once
npx --yes playwright screenshot --viewport-size=1200,630 og-template.html public/og-default.png
```

The template pulls `schemes.css` + `tokens.css` so the OG image is literally on-token; the logo is an exact-file copy. Wire it in the head component (`og:image` absolute URL, `og:image:width` 1200, `og:image:height` 630 — the head component itself belongs to seo-aeo-schema). Verify the PNG visually before shipping.

## Limitations

- This document does NOT cover which imagery may appear on the site at all (the canon's G-IMG rules and the brand's image bank govern that) or the SEO head component (seo-aeo-schema).
- Command recipes (`sharp-cli`, `playwright screenshot`) are one verified path; any SVG rasterizer / headless browser works — the constraints (raster, on-system, exact-file) are the contract, the tool is not.

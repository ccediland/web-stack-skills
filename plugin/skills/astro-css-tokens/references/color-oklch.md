# Color — OKLCH

Load when debugging color output or deciding on a fallback strategy.

## Why OKLCH

OKLCH (Oklab Lightness Chroma Hue) is perceptually uniform: equal numerical steps produce equal perceived lightness changes across hues. sRGB and HSL are not uniform — a `blue` at L=50 looks much lighter than a `red` at L=50. OKLCH makes palette interpolation predictable and eliminates the "muddy middle" problem in color scales.

Tailwind v4 ships its entire default palette in OKLCH. Authoring tokens in OKLCH keeps the palette consistent with Tailwind's system and avoids round-trip conversion loss.

## No fallback

OKLCH (`oklch()`) is Baseline Widely Available as of 2025-11-09:

| Browser | Supported since |
|---|---|
| Chrome / Edge | 111 (Mar 2023) |
| Firefox | 113 (May 2023) |
| Safari | 15.4 (Mar 2022) |

No `@supports` guard or `rgb()` fallback is needed. The minimum supported browser in the target stack predates all three milestones.

## Preserving OKLCH through Style Dictionary

SD's `color/css` transform passes token values through [Color.js](https://colorjs.io), which gamut-maps to sRGB and emits hex or `rgb()`. OKLCH is lost.

To preserve OKLCH strings unchanged, use only name transforms:

```javascript
transforms: ['attribute/cti', 'name/kebab'],
```

Never add `color/css` or include a transform group that contains it (e.g., `transformGroup: 'css'` includes `color/css`).

SD v5.3+ ships a `color/oklch` transform that converts structured DTCG color objects to OKLCH strings — but this skill authors `$value` as a plain OKLCH string from the start, so the transform is unnecessary.

## Legacy escape hatch

If a consumer downstream cannot handle `oklch()` (e.g., an email renderer, a PDF export library), define a separate SD platform in `build.mjs` that adds `color/css` and writes to a different `buildPath`. Never mix it into the main CSS platform.

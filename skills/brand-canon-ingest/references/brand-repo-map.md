---
title: Brand repo consumption map
summary: Anatomy of a contract-0.6.0 brand repo from the consumer's seat — what the site copies verbatim, generates from, loads, or must never touch; the cubicBezier transform; JS-vocabulary token import.
last_updated: 2026-08-17
applies_to: "brand-system-skills 0.6.0 · style-dictionary@5.5.1"
---

# Brand repo consumption map

Load when first opening a brand repo, deciding what a given file is for, or wiring the two token-consumption seams (cubicBezier, JS vocabulary).

## Anatomy — the consumer's view

Every repo the brand-canon-builder emits carries the same load-bearing surfaces. The site's verb per surface:

| Surface | What it is | Site verb |
|---|---|---|
| `tools/run-gates.mjs` | the brand's self-enforcing gate board (the contract check) | RUN first — green or stop |
| `tokens/web/{base,semantic,component}.json` | pre-emitted SD-ready string projection (plain `oklch()` strings, provenance stripped, no hex) | COPY verbatim into the site's `tokens/` |
| `tokens/*.tokens.json` (the spine) | source of truth, structured-OKLCH `$value` objects | NEVER consume directly |
| `tokens/schemes/*.tokens.json` | materialized scheme role sets, structured-OKLCH | GENERATE `schemes.css` via the vendored serializer |
| `tools/tokens-project.mjs` | the brand's own projector spine → `tokens/web/` | RUN in the brand repo if `tokens/web/` is missing or stale |
| `assets/logos/` · `assets/icons/` · `assets/fonts/` | exact-file brand assets | COPY exact-file (see assets-fonts-og.md) |
| `canon/canon.json` + `<brand>-keystone.md` + `<brand>-visual-keystone.md` | the machine mirror + the verbal and visual brains (resident set) | LOAD together for copy and design judgment |
| `satellites/asset-index.md` | the one consultation map of every asset and knowledge doc | LOOK UP before hunting files by hand |
| `satellites/data-map.md` | where each volatile datum lives | POINT the site at those sources |
| `satellites/projections.md` | registry of every consumer | REGISTER the site here |
| `canon/0*.md` (layer docs) · `sources/` · `audit/` | canon prose, custody, evidence | READ when a rule needs context; never edit |

Nothing in the brand repo is ever edited from the site side. A needed change is builder territory: edit the root there, re-run its tools and gates, then re-ingest here.

## Why tokens/web/ and never the spine

The spine keeps color `$value` as a structured object (DTCG 2025.10) so provenance and multi-space values survive. Style Dictionary v5 splits object `$value` into `-value`/`-unit` sub-vars (bugs #1398 — fixed for dimensions, #1494 — still open for composites), and the stack's contract (astro-css-tokens) requires plain-string `$value`. The brand repo resolves this itself: `tools/tokens-project.mjs` emits `tokens/web/` — the same tree with every color serialized to the canonical `oklch()` string, `$extensions` provenance dropped, hex fallbacks not propagated, custody recorded in the brand's `sources/MANIFEST.json`. The site consumes that projection as-is. Regeneration always happens in the BRAND repo so custody stays true.

Two properties of the copied files to leave alone:

- Group-level `$type` — the files set `$type` on groups (`base.color`, `base.spacing`) and inherit to leaves. Valid DTCG; SD v5 with `usesDtcg: true` resolves it. Do not add per-leaf `$type`; the files stay byte-identical to the brand repo so drift is grep-detectable.
- Aliases are `{tier.category.name}` pointing at leaves — exactly what the astro-css-tokens pipeline expects. `outputReferences: true` keeps them as `var()` chains.

## The one pipeline extension — cubicBezier

Easing tokens are DTCG cubicBezier arrays (`"$value": [0.22, 0.61, 0.36, 1]`). Name-only transforms would emit them raw — invalid CSS. Register one value transform in the site's `build.mjs` (this is the sanctioned consumer-side adapter; it does not modify the astro-css-tokens reference config, it adds to it):

```javascript
StyleDictionary.registerTransform({
  name: 'value/cubic-bezier-css',
  type: 'value',
  transitive: true,
  filter: (token) => token.$type === 'cubicBezier' && Array.isArray(token.$value),
  transform: (token) => `cubic-bezier(${token.$value.join(', ')})`,
});
```

Then add `'value/cubic-bezier-css'` to the platform's `transforms` array next to `attribute/cti` and `name/kebab`. Result: `--ds-base-motion-easing-soft: cubic-bezier(0.22, 0.61, 0.36, 1);`.

## JS-vocabulary tokens — import the JSON, not the CSS var

Part of the motion vocabulary is not CSS at all: unitless scales (`press-scale: 0.98`), observer config (`reveal-threshold: 0.12`, `reveal-root-margin: "0px 0px -8% 0px"`), rate values (`marquee-autonomous-speed: "78px/s"`). Consume them in script by importing the copied token JSON — Vite resolves JSON imports natively:

```javascript
import base from '../../tokens/base.json';

const vocab = base.base.motion.vocabulary;
const observer = new IntersectionObserver(onReveal, {
  threshold: Number(vocab['reveal-threshold'].$value),
  rootMargin: vocab['reveal-root-margin'].$value,
});
```

Emitting these as CSS vars is harmless but useless — the consumers are JS APIs. Rates like `78px/s` are brand vocabulary the animation code interprets (px per second), not CSS values.

## Contract pin discipline

Record in the site (its dev doc or decision log) which contract the ingest ran against: brand-system-skills plugin version + tool-repo commit, and the brand repo commit consumed. Emitted repos carry no version stamp by design — the gates enforce the contract behaviorally — so the pin lives on the consumer side. On a newer builder release, re-check the four load-bearing surfaces named in the SKILL.md review-gate before ingesting repos it emitted.

## Limitations

- This document does NOT cover SD configuration itself (astro-css-tokens `references/style-dictionary-config.md`) or scheme serialization (`scheme-serializer.md`).
- The anatomy table lists contract surfaces; a given brand repo may carry extra brand-specific directories (prototypes, design-sync kits) that the site does not consume.

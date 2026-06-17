# Lighthouse CI Configuration

> Full lighthouserc.json and budget.json for an Astro 6 plus Cloudflare Workers site: collection targets, assertion presets and tiers, run count, and the absolute-base trap. Sizes and metrics here are regression ceilings, not Google field thresholds.

## Table of contents
- Collection target
- The absolute-base trap
- Assertion presets
- Assertion tiers
- Desktop variant
- budget.json
- Run count and stability

## Collection target

LHCI must serve the built site somehow before it can audit it. Two paths.

### staticDistDir (default for this stack)

```json
"collect": { "staticDistDir": "./dist/client", "numberOfRuns": 5 }
```

LHCI spins up its own static file server against the directory and audits every HTML file it finds. Most reproducible, no runtime needed. The Cloudflare adapter defaults to `output: 'server'`, so prerendered client assets are written to `dist/client`, not `dist`; pointing at `./dist` finds nothing. This path does not serve the adapter's immutable cache headers, which does not matter for a regression gate.

### startServerCommand with astro preview (SSR routes only)

```json
"collect": {
  "startServerCommand": "npm run preview",
  "url": ["http://localhost:4321/", "http://localhost:4321/about"],
  "startServerReadyPattern": "localhost",
  "numberOfRuns": 5
}
```

In Astro 6 with `@astrojs/cloudflare` v13, `astro preview` runs on the real Cloudflare workerd runtime, so this audits on-demand routes against a production-parity server. It is heavier and slower in CI than staticDistDir; use it only when SSR routes must be audited. List explicit `url` entries because there is no static directory to crawl.

## The absolute-base trap

Do not set an absolute `site` together with `base` on the build you audit. The Cloudflare build writes assets to `dist/client/*` without the base prefix, so a literal server (the staticDistDir server, or a localhost port) returns 404 for `_astro/*` and CSS fails to load, tanking the score for the wrong reason. Either drop `base` for the audited build, or use the preview-server path which routes the base correctly.

## Assertion presets

| Preset | Effect |
|---|---|
| lighthouse:all | every audit must be perfect; far too strict for a real gate |
| lighthouse:recommended | non-performance audits must pass, metrics warn below ~0.9, flaky audits are not hard-failed |
| lighthouse:no-pwa | recommended minus PWA audits |

The PWA category was removed in Lighthouse 12, so no-pwa and recommended are nearly equivalent. Prefer no-pwa to guarantee no stale PWA assertion fires.

## Assertion tiers

Assertions are ESLint-style: `"audit-id": [level, options]`, where level is off, warn, or error, and options are commonly `{ "minScore": n }` for categories or `{ "maxNumericValue": n }` for metrics. Metric numeric values are in milliseconds, or unitless for CLS. The default option set is `{ "aggregationMethod": "optimistic", "minScore": 1 }`.

Gate as error only the metrics that are stable and meaningful in the lab; gate diagnostics as warn so they surface without blocking on noise.

```json
"assert": {
  "preset": "lighthouse:no-pwa",
  "assertions": {
    "categories:performance": ["error", { "minScore": 0.9 }],
    "categories:accessibility": ["error", { "minScore": 0.95 }],
    "categories:best-practices": ["warn", { "minScore": 0.9 }],
    "categories:seo": ["warn", { "minScore": 0.9 }],
    "largest-contentful-paint": ["error", { "maxNumericValue": 2500 }],
    "total-blocking-time": ["error", { "maxNumericValue": 300 }],
    "cumulative-layout-shift": ["error", { "maxNumericValue": 0.1 }],
    "first-contentful-paint": ["warn", { "maxNumericValue": 2000 }],
    "speed-index": ["warn", { "maxNumericValue": 3400 }],
    "interactive": ["warn", { "maxNumericValue": 3800 }],
    "uses-long-cache-ttl": "off",
    "uses-http2": "off"
  },
  "budgetsFile": "budget.json"
}
```

Performance and accessibility are errors because they are the load-bearing floors with no other hard gate in the seven skills until the SEO and visuals skills land. Best-practices and SEO are warnings so the seo-aeo-schema skill owns the hard SEO gate. Total Blocking Time stands in for INP, which the lab cannot measure. `uses-long-cache-ttl` is turned off because the staticDistDir server does not emit production cache headers, so the audit would always fail there.

## Desktop variant

Mobile is the Lighthouse default (a Moto-G-class device with Slow-4G throttling and a 4x CPU slowdown). To gate desktop as well, add a second run with the desktop preset and stricter ceilings.

```json
"collect": {
  "staticDistDir": "./dist/client",
  "numberOfRuns": 5,
  "settings": { "preset": "desktop" }
}
```

Desktop ceilings should be tighter: LCP 2000, TBT 200, FCP 1500, Speed Index 2600, TTI 2900. Running both mobile and desktop yields only one GitHub status report, so if both matter, run them as two jobs or two workflow steps and read the second report from the run logs. The budget table for both form factors is in budgets-and-thresholds.md.

## budget.json

Resource budgets are a separate file referenced by `budgetsFile`. Sizes are in kilobytes (note the unit difference: assertion `maxNumericValue` uses bytes, budget.json uses kilobytes). They fail the build when a page exceeds the byte weight or count for a resource type.

```json
[
  {
    "path": "/*",
    "resourceSizes": [
      { "resourceType": "script", "budget": 150 },
      { "resourceType": "stylesheet", "budget": 60 },
      { "resourceType": "image", "budget": 400 },
      { "resourceType": "font", "budget": 100 },
      { "resourceType": "total", "budget": 800 }
    ],
    "resourceCounts": [
      { "resourceType": "third-party", "budget": 5 },
      { "resourceType": "total", "budget": 30 }
    ]
  }
]
```

Per-route budgets are possible by adding objects with narrower `path` globs (for example a heavier budget on `/gallery/*`). Keep the `/*` default conservative for a premium marketing site; the visuals skills (WebGL, Rive, motion) push script and image weight, so tune these budgets once those land rather than fighting them per commit.

## Run count and stability

Set `numberOfRuns: 5`. LHCI reports the median, and the median of 5 runs is roughly twice as stable as a single run; even 3 runs cuts variance materially. Shared CI runners add variance, so prefer warn over error for diagnostics and reserve error for the metrics with real headroom above current measurements.

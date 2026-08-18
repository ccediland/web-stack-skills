---
title: Prefetching and speculation — the navigation-speed recipe
summary: The default (Astro's stable prefetch), the escalation (manual Speculation Rules prerender with the analytics guard), why experimental.clientPrerender is not taught, and why Cloudflare Speed Brain does not apply to this stack — added in W4.
last_updated: 2026-08-18
applies_to: astro@7.2.2 prefetch (stable) · Speculation Rules API (NOT Baseline — Chromium-real) · Workers Static Assets (Speed Brain excluded)
---

# Prefetching and speculation

> This is a RECIPE because the default is one config line and the escalation is one JSON block plus one guard — no per-site synthesis. The non-obvious content is what NOT to do: do not teach the experimental flag, do not expect Speed Brain to help on Workers, and do not let browser prerender double-count the analytics layer this playbook just shipped.

## Contents

- The default — Astro prefetch
- The escalation — manual Speculation Rules
- The analytics guard (mandatory with prerender)
- What is deliberately NOT used
- CSP note
- Review-gates

## The default — Astro prefetch

```js
// astro.config.mjs
export default defineConfig({
  prefetch: { prefetchAll: false, defaultStrategy: 'hover' },
});
```

Stable, cross-browser, cheap: `data-astro-prefetch` on the links that deserve it (primary nav, the conversion page), `hover` default (`tap` for mobile-heavy audiences, `viewport` for a short obvious next step). Under the hood it is `link rel="prefetch"` with a `fetch()` fallback — body-only, NO JS execution, no analytics interaction, works everywhere. For a mostly-static site on fast edge hosting this captures most of the perceived win; it is the entire recipe for most briefs.

## The escalation — manual Speculation Rules

When the brief demands INSTANT next-page paint (not just a warm cache) and accepts Chromium-only enhancement:

```html
<script type="speculationrules">
  {
    "prerender": [{
      "where": { "and": [
        { "href_matches": "/*" },
        { "not": { "selector_matches": ".no-prerender" } }
      ]},
      "eagerness": "moderate"
    }]
  }
</script>
```

- NOT Baseline: real in Chrome/Edge; Firefox position neutral; Safari has no stable ship. Firefox and Safari users get normal navigation — that is the designed behavior, never a bug to fix.
- `moderate` eagerness holds at most 2 prerenders (immediate/eager allow 10, FIFO-evicted); the browser auto-disables speculation under Save-Data, low memory, or battery saver.
- Prerender fetches and FULLY EXECUTES the page in a hidden renderer — which is exactly why the guard below is not optional.
- Exclude unsafe-to-speculate URLs by class or pattern: anything with side effects on load. On this stack that is at minimum `/admin/` and any future logout/one-time-token URL; `/_actions/*` endpoints are not links and never speculate.

## The analytics guard (mandatory with prerender)

Browser prerender re-executes page JS — including the analytics script and any exposure events — before the visitor ever sees the page. Unguarded, every prerender double-counts a pageview and fires `ab_expose` for pages never viewed. The guard, in the layout's analytics/experiment script:

```js
function initMeasurement() {
  // analytics init + exposure events live here
}
if (document.prerendering) {
  document.addEventListener('prerenderingchange', initMeasurement, { once: true });
} else {
  initMeasurement();
}
```

`prerenderingchange` fires once when the hidden page is activated into the real tab. Retroactive check where needed: a non-zero `activationStart` on the navigation timing entry means this pageview was a prerender activation. Server-side, speculative requests carry `Sec-Purpose: prefetch` — relevant only if an on-demand route ever needs to skip side effects. Rule of composition: the escalation is INSEPARABLE from this guard — a site adopts both in the same commit or neither.

## What is deliberately NOT used

- `experimental.clientPrerender` — Astro's flag that emits Speculation Rules from the prefetch config. It is still EXPERIMENTAL in 7.2.2, and this catalog does not teach experimental APIs (standing rule). When it stabilizes, it replaces the manual block above — that is its review-gate, not a reason to enable it early.
- Cloudflare Speed Brain — the platform's own Speculation-Rules injection. Its docs state verbatim that it "will not prefetch on routes that run Workers": on Workers Static Assets every route is served through a Worker, so Speed Brain does not apply to this stack. Do not enable it expecting effect; do not count on it in perf planning.
- Prerendering the conversion page from everywhere — the contact page loads Turnstile; prerendering it burns challenge tokens and skews the funnel. Prefetch (body-only) is fine for it; prerender is not.

## CSP note

The inline `speculationrules` script block is NOT executable JS but IS governed by script-src. Under Astro's hash CSP, give it the single-sourced manual-hash treatment (the NO_FLASH/AB_SCRIPT pattern — one constant, hashed in config, rendered in the layout), or serve the rules via the `Speculation-Rules` HTTP header to skip CSP entirely (a `_headers` line on this stack — coordinate with web-security-headers' file).

## Review-gates

| Watch | Today (2026-08-18) | Flip |
|---|---|---|
| Speculation Rules Baseline | NOT Baseline; Firefox neutral, Safari prefetch-only signals | Baseline arrival → escalation stops being Chromium-only enhancement |
| `experimental.clientPrerender` | experimental in astro 7.2.2 | stabilization → replaces the manual block, one config line |
| Speed Brain × Workers | excluded on Worker-served routes, beta | Cloudflare lifting the exclusion → re-evaluate the whole recipe |
| Chrome limits | 10 prerender / 50 prefetch (immediate), 2/2 (moderate+) | limits move with the API |

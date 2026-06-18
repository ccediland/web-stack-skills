---
title: CSP and reduced-motion per engine
summary: The two cross-cutting, gate-relevant concerns — a ship-safe Content-Security-Policy recipe for each motion engine under Astro 6 hash-based meta CSP (script loading and runtime style writes), and the prefers-reduced-motion recipe for each engine tied to the perf-ci-gates 0.95 accessibility floor.
last_updated: 2026-06-17
applies_to: astro@6.4.7 security.csp, @astrojs/cloudflare@13.7.0, gsap@3.15.0, motion@12.40.0
---

# CSP and reduced-motion per engine

> Two concerns that cut across all three engines and both gate against sibling skills. CSP composes with web-security-headers; reduced-motion composes with the perf-ci-gates accessibility floor of 0.95.

## Contents
- How Astro CSP reaches motion
- Script loading rule
- Runtime style writes
- Ship-safe recipe per engine
- prefers-reduced-motion per engine
- Limitations

## How Astro CSP reaches motion
On Astro 6 with the Cloudflare adapter (no staticHeaders), the CSP is delivered as a hash-based meta http-equiv tag, not a response header. Astro emits hashes for all the scripts it bundles, including client islands, so anything Astro bundles passes a strict script-src automatically. Two distinct questions decide whether motion survives that policy: how the engine script is loaded, and what the engine does to styles at runtime.

## Script loading rule
Load every engine via a bundled Astro script or a hydrated island, never is:inline. Astro processes and hashes bundled scripts and island hydration code, so they satisfy strict script-src. An is:inline script is not processed or hashed and is blocked under the strict policy, because unsafe-inline is incompatible with Astro CSP and browsers ignore unsafe-inline when a hash is present. External CDN scripts are not hashed out of the box: self-host via an npm import (the engines are installed as dependencies), or supply your own hash in security.csp.scriptDirective.hashes.

## Runtime style writes
GSAP and Motion animate by writing per-property inline styles through CSSOM, for example element.style.transform. Per the CSP specification, styles set directly on an element style property are NOT blocked by style-src, so JavaScript can safely manipulate styles this way; only styles applied via setAttribute style or element.style.cssText are governed by style-src and style-src-attr. GSAP and Motion per-property writes and WAAPI animations therefore run under a hash-based style-src with no unsafe-inline and no unsafe-hashes. Avoid bulk cssText and setAttribute style writes under strict CSP. CSS scroll-driven animations are pure declarative CSS and have zero CSP interaction.

## Ship-safe recipe per engine

| Engine | Loading | CSP outcome |
|---|---|---|
| CSS scroll-driven | stylesheet or hashed style block | no script-src concern; declarative styles only; zero special directives |
| GSAP vanilla | bundled Astro script, not is:inline | script auto-hashed, passes strict script-src; runtime element.style writes exempt from style-src |
| GSAP in island | island hydrated client:visible or client:idle | hydration code auto-hashed; same CSSOM exemption |
| Motion in island | import from motion/react inside the island | island script auto-hashed; WAAPI and element.style writes exempt |

No unsafe-inline or unsafe-hashes is required for any engine. CSP is testable in build and preview, not the dev server, so validate the policy against a built site.

## prefers-reduced-motion per engine

Reduced-motion handling is gate-relevant, not optional: the perf-ci-gates accessibility floor is 0.95 and missing reduced-motion handling is flagged under WCAG 2.3.3.

CSS scroll-driven: gate the animation inside no-preference and leave a static, readable default.

```css
.reveal { opacity: 1; transform: none; }
@supports (animation-timeline: view()) {
  @media (prefers-reduced-motion: no-preference) {
    .reveal { animation: fade-up linear both; animation-timeline: view(); animation-range: entry 0% entry 100%; }
  }
}
```

GSAP: use gsap.matchMedia with a no-preference condition; the reduce branch builds nothing.

```js
const mm = gsap.matchMedia();
mm.add("(prefers-reduced-motion: no-preference)", () => {
  gsap.to(".panel", { scrollTrigger: { trigger: ".panel", scrub: true }, x: 200 });
});
```

Motion: use the useReducedMotion hook to branch, or wrap in MotionConfig reducedMotion user to disable transform and layout animations while keeping opacity.

```jsx
import { useReducedMotion } from "motion/react";
const reduce = useReducedMotion();
// if reduce, render static or opacity-only motion
```

## Limitations & Out-of-Scope
- The CSSOM-exemption holds for per-property element.style writes; cssText and setAttribute style are governed. Validate the chosen engine write path in a CSP meta staging build before locking.
- Astro CSP does not run in the dev server; verify in build and preview.
- Does NOT cover nonce-based CSP or SSR header CSP; the Cloudflare adapter path here is meta only.

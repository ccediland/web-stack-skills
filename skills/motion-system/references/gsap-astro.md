---
title: GSAP plus ScrollTrigger on Astro 6
summary: Idiomatic loading on Astro 6 plus Cloudflare — bundled script for static pages versus useGSAP inside a React island, SSR-safety, tree-shaking imports, the lazy off-critical-path pattern, verified bundle weight, and what ScrollTrigger pin and scrub do.
last_updated: 2026-06-17
applies_to: gsap@3.15.0, @gsap/react@2.1.2, astro@6.4.7, @astrojs/cloudflare@13.7.0
---

# GSAP plus ScrollTrigger on Astro 6

> Use GSAP only for scrub, pin, snap, nested orchestration, or motion paths. Load it via a bundled Astro script on static pages or the useGSAP hook inside a React island, register ScrollTrigger client-side, and lazy-load it off the critical path. GSAP and ScrollTrigger are client-only.

## Contents
- License and weight
- Tree-shaking imports
- Static page: bundled script
- React island: useGSAP
- SSR-safety
- Lazy off the critical path
- What ScrollTrigger adds
- prefers-reduced-motion
- Limitations

## License and weight
GSAP is 100 percent free including ScrollTrigger and SplitText for commercial use since April 2025 (Webflow). The older claim that SplitText needs a Club membership is false. Verified weights for the perf-ci-gates budget: gsap core about 27 KB gzipped, the ScrollTrigger plugin about 18 KB gzipped, so a page using both budgets at about 45 KB gzipped. Treat the ScrollTrigger figure as a single-source estimate and confirm against the real compressed build.

## Tree-shaking imports
Import the core and only the plugins you use, by per-plugin path, not gsap/all.

```js
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
gsap.registerPlugin(ScrollTrigger);
```

## Static page: bundled script
On a static page with no React, use a bundled Astro script (a script tag with no is:inline). Astro processes it: TypeScript, import bundling, type module, and CSP hashing. The code runs only in the browser, so importing gsap and ScrollTrigger at the top is safe.

```astro
<script>
  import gsap from "gsap";
  import { ScrollTrigger } from "gsap/ScrollTrigger";
  gsap.registerPlugin(ScrollTrigger);
  gsap.to(".panel", {
    scrollTrigger: { trigger: ".panel", start: "top center", scrub: true },
    x: 200,
  });
</script>
```

This is preferable to an island for non-React motion because it ships no React runtime, which fits Astro zero-JS-by-default.

## React island: useGSAP
Inside a React island, use the useGSAP hook from @gsap/react. It is a drop-in replacement for useEffect and useLayoutEffect that runs animation setup and auto-reverts every tween, ScrollTrigger, and SplitText created during the hook when the component unmounts, via gsap.context. It is SSR-safe because it prefers useLayoutEffect and falls back to useEffect when window is undefined.

```jsx
import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { useGSAP } from "@gsap/react";
import { useRef } from "react";

gsap.registerPlugin(ScrollTrigger, useGSAP);

export default function Scene() {
  const root = useRef(null);
  useGSAP(() => {
    gsap.to(".item", { scrollTrigger: ".item", y: 0, opacity: 1, stagger: 0.1 });
  }, { scope: root });
  return <section ref={root}>...</section>;
}
```

Wrap event handlers and timeouts created after setup in contextSafe so they join the context and get cleaned up. The hook only runs after the island hydrates, so it composes with client directives.

## SSR-safety
Never call ScrollTrigger code in .astro frontmatter or a React render body: the server has no document and will throw document is not defined. Keep all GSAP execution inside a bundled script, the useGSAP hook, or a dynamic import that runs client-side.

## Lazy off the critical path
Load GSAP only when the scene is near, to protect the perf budget.

```js
const io = new IntersectionObserver(async ([entry]) => {
  if (!entry.isIntersecting) return;
  io.disconnect();
  const { gsap } = await import("gsap");
  const { ScrollTrigger } = await import("gsap/ScrollTrigger");
  gsap.registerPlugin(ScrollTrigger);
  // build the scene
});
io.observe(document.querySelector(".scene"));
```

Inside an island, a client:visible directive achieves the same: the island bundle loads only when it scrolls near.

## What ScrollTrigger adds
The things CSS scroll-driven cannot do: pin with content lock (pin true plus scrub, which creates a pin-spacer and uses position fixed), snap to panels, scrubbed multi-element timelines, onEnter and onUpdate callbacks with self.progress, and motion paths via MotionPathPlugin. Call ScrollTrigger.refresh() after layout, content, or image changes that shift positions.

## prefers-reduced-motion
Gate all setup with gsap.matchMedia so it auto-reverts when the query stops matching.

```js
const mm = gsap.matchMedia();
mm.add("(prefers-reduced-motion: no-preference)", () => {
  gsap.to(".panel", { scrollTrigger: { trigger: ".panel", scrub: true }, x: 200 });
});
```

The reduce branch simply builds nothing, leaving content in its static state. This protects the perf-ci-gates accessibility floor of 0.95.

## Limitations & Out-of-Scope
- Client-only; no SSR execution.
- Weight is non-trivial; always lazy-load and keep under the budget. If the page JS budget cannot absorb about 45 KB, fall back to CSS scroll-driven or Motion Mini.
- Does NOT replace native CSS for simple reveals; reserve GSAP for scrub, pin, snap, orchestration, or motion paths.

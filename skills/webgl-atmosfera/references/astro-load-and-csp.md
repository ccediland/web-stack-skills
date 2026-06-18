---
title: Astro load and CSP for the shader hero
summary: Why the canvas is never the LCP element and what that means, the correct lazy-init path for an above-the-fold canvas (requestIdleCallback plus IntersectionObserver, not client:visible), how to reserve the canvas box against CLS, and how the bundled shader script earns its place in a hash-based CSP.
last_updated: 2026-06-18
applies_to: Astro 6 islands and CSP; Cloudflare; Core Web Vitals
---

# Astro Load and CSP for the Shader Hero

> The hero must paint and become interactive before the shader exists. The shader is an enhancement layered on afterward, wired so it cannot move the LCP, shift layout, or need a CSP exception.

## Contents

- The canvas is not the LCP element
- The wrong directive and the right init
- Reserving the box against CLS
- CSP parity with bundled scripts
- The Astro island shape

## The canvas is not the LCP element

The set of elements eligible to be the Largest Contentful Paint element is deliberately limited to images, video poster frames, elements with a background image loaded through the url function, and block-level elements containing text. A canvas is not in that set, and CSS gradients are explicitly excluded. The consequence is load-bearing: the hero's LCP element will be the headline text or a foreground image, never the shader canvas or a gradient fallback. So the headline or hero image must render from static HTML immediately and must not depend on the shader initializing. This is also why a fade-in of the canvas via opacity is safe — the canvas is not the LCP element, so revealing it late costs nothing on that metric, and opacity is a compositor-only property.

## The wrong directive and the right init

For an above-the-fold element, Astro's client:visible directive is the wrong tool. It hydrates on IntersectionObserver entry, but an above-the-fold element is already in the viewport on load, so the observer fires immediately and gives no deferral. client:load is also wrong, because it competes with the critical render. The right path is to initialize manually inside a small first-party script: wait out the critical render with requestIdleCallback, then gate the actual start on real visibility with IntersectionObserver. The manual path is preferred over the client:idle directive because the IntersectionObserver logic is needed anyway for the runtime pausing in the perf reference, so it is one mechanism rather than two.

```js
function startWhenIdleAndVisible(canvas, init) {
  const ric = window.requestIdleCallback || ((cb) => setTimeout(cb, 200));
  ric(() => {
    const io = new IntersectionObserver((entries, obs) => {
      if (entries[0].isIntersecting) {
        obs.disconnect();
        init(); // create context, start the loop
      }
    });
    io.observe(canvas);
  });
}
```

## Reserving the box against CLS

Cumulative Layout Shift goes to zero only if the canvas occupies stable space before the context initializes. Give the canvas or its container an explicit size or aspect-ratio in CSS so layout is fixed from first paint. Size the drawing buffer separately and later — canvas.width and canvas.height multiplied by the clamped device pixel ratio, set on resize — which is independent of the laid-out CSS box. Never insert the canvas late or resize the laid-out box after content has painted around it.

```css
.hero-atmosphere {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
  opacity: 0;
  transition: opacity 600ms ease;
}
.hero-atmosphere.is-ready { opacity: 1; }
```

## CSP parity with bundled scripts

Astro 6 emits a Content-Security-Policy in which every script it bundles, including client islands, is represented by a SHA hash in script-src; modern browsers reject unsafe-inline once a hash or nonce is present in the directive. This is the same mechanism by which a bundled motion library's script is hashed in the sibling security-headers and motion skills, so the shader module inherits it for free. Four consequences follow:

- A first-party WebGL module bundled by Astro is hash-emitted into script-src automatically. No manual hashing, no exception. This is full parity with how a bundled motion library is handled.
- Author shaders as JS template-literal strings inside the bundled module so nothing is fetched at runtime — no .glsl file load, so no connect-src directive is involved.
- The WebGL canvas rendering context is not governed by CSP at all. CSP governs resource loading, not GPU draw calls or shader compilation, so getContext and drawArrays need no directive.
- worker-src is only relevant if rendering moves into a Worker via OffscreenCanvas. The default main-thread canvas spawns no worker and needs no worker-src.

Enable CSP through the Astro config; the script directive carries the hashes.

```js
// astro.config.mjs
export default {
  experimental: {
    csp: {
      scriptDirective: { hashes: [] }, // Astro fills bundled-script hashes; add cross-origin hashes here if any
    },
  },
};
```

## The Astro island shape

Render the canvas in a static .astro component so the markup is server-rendered and the box is reserved immediately, then attach the first-party init script. Do not hydrate it as a framework island; there is no component state to hydrate, only an imperative WebGL loop, so a plain script bundled by Astro is lighter and keeps the CSP story simple.

## Limitations

- This assumes the hero headline or image is the LCP element and renders from static HTML. If a design makes the canvas visually dominant with no text or image LCP candidate, revisit — a slow LCP cannot be fixed by deferring the canvas, because the canvas was never the LCP element.
- CSP behavior is verified against Astro 6's hash-based script directive; a future major may change the config surface, so re-check on upgrade.

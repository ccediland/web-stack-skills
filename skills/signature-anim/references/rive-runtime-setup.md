---
title: Rive runtime setup, lazy init, and lifecycle in Astro
summary: Renderer choice and measured sizes, a vanilla TypeScript IntersectionObserver lazy-init that keeps the heavy WASM off the critical path, and the offscreen-pause and cleanup lifecycle.
last_updated: 2026-06-18
applies_to: astro@6.4.7, @rive-app/canvas@2.38.1, Node 22
---

# Rive runtime setup, lazy init, and lifecycle in Astro

The Rive runtime plus WASM is the single heaviest asset in this stack, so the goal of setup is to keep it off the critical render path and to tear it down cleanly. Use a plain TypeScript module, not a framework island, for a single non-React site.

## Contents

- Renderer choice and sizes
- Why vanilla, not an island
- Reserve the box first
- Lazy init with IntersectionObserver
- Offscreen and hidden-tab pause
- Teardown and cleanup
- Coexisting with a WebGL background

## Renderer choice and sizes

Default to `@rive-app/canvas` (Canvas2D). It has no browser WebGL context limit, renders vector, raster, and mesh deformations, and is the recommended runtime. Reach for `@rive-app/webgl2` only when the Rive Renderer, heavy mesh, or maximum edit-time fidelity is required; it consumes a WebGL context. Never `@rive-app/webgl` (legacy).

Measured 2026-06-18; re-measure at build because versions churn weekly.

| Package | JS gzip | rive.wasm raw | Combined transfer |
|---|---|---|---|
| @rive-app/canvas | about 42 KB | about 1.93 MB | about 840 KB gzip |
| @rive-app/canvas-lite | about 35 KB | about 806 KB | about 390 KB gzip |
| @rive-app/webgl2 | about 44 KB | about 2.43 MB | about 1.05 MB gzip |

The WASM, not the JS, dominates. canvas-lite drops Rive Text, audio, and scripting and roughly halves the WASM; use it when the `.riv` needs none of those. The widely cited 200 KB figure is outdated. The compressed served size is version-volatile; measure it and set the perf budget against the measured number, treating the WASM as a separate long-cached resource excluded from the critical budget.

## Why vanilla, not an island

A framework island would ship a framework runtime and risk above-the-fold hydration. A single signature Rive moment on a non-React site is best a plain TypeScript module with manual init. The inline bootstrap is hashed by Astro CSP automatically. Use `@rive-app/react-canvas` and useRive only when the page is already a React island.

## Reserve the box first

Give the canvas explicit dimensions or an aspect-ratio in CSS so lazy init causes no layout shift, protecting CLS.

```html
<canvas id="hero-canvas" style="width:100%; aspect-ratio:16/9;"></canvas>
```

A canvas is not LCP-eligible; LCP candidates are text, images, video posters, and background images. Deferring the canvas therefore does not hurt LCP. Make the LCP element a hero heading or image, not the Rive canvas.

## Lazy init with IntersectionObserver

Dynamic-import the runtime, the WASM URL, and the `.riv` only when the canvas nears the viewport, never as a top-level import or an above-the-fold island.

```ts
const canvas = document.getElementById('hero-canvas') as HTMLCanvasElement;
let rive: any = null;

const io = new IntersectionObserver(async (entries) => {
  const e = entries[0];
  if (e.isIntersecting && !rive) {
    const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
    try {
      const { Rive, RuntimeLoader } = await import('@rive-app/canvas');
      const wasmUrl = (await import('@rive-app/canvas/rive.wasm?url')).default;
      RuntimeLoader.setWasmUrl(wasmUrl);
      const src = (await import('./hero.riv?url')).default;
      rive = new Rive({
        src, canvas,
        stateMachines: 'Hero',
        autoplay: !reduce,
        automaticallyHandleEvents: false,
        onLoad: () => rive.resizeDrawingSurfaceToCanvas(),
      });
    } catch {
      // leave the static poster in place, see a11y-and-fallback.md
    }
  } else if (rive) {
    e.isIntersecting ? rive.play() : rive.pause();
  }
}, { rootMargin: '200px' });

io.observe(canvas);
```

Supply `stateMachines` or Rive plays the first linear animation instead of the state machine. Call `resizeDrawingSurfaceToCanvas()` in onLoad and again on window resize for crisp device-pixel-ratio rendering.

## Offscreen and hidden-tab pause

Pause when the canvas leaves the viewport (the observer branch above) and when the tab is hidden. Rive also self-pauses when the state machine is idle with no looping or blend states active, so an interaction-gated moment costs almost no CPU at rest.

```ts
document.addEventListener('visibilitychange', () => {
  if (!rive) return;
  document.hidden ? rive.pause() : rive.play();
});
```

Use pause and play, or stopRendering and startRendering to halt the render loop without changing play state.

## Teardown and cleanup

Rive creates C++ and WASM-backed objects that must be freed, or they leak. Call cleanup on teardown.

```ts
window.addEventListener('beforeunload', () => rive?.cleanup());
```

In a single-page-app context, call cleanup when the view holding the canvas unmounts. For the Canvas2D runtime cleanup is sufficient; the WebGL runtime additionally needs deleteRiveRenderer.

## Coexisting with a WebGL background

When a sibling skill renders a fullscreen WebGL shader background on the same page, use the Canvas2D `@rive-app/canvas` for Rive so it does not consume one of the browser's limited WebGL contexts (Chrome allows about 16 on desktop and 8 on Android, per page or site; exceeding the limit loses the oldest context). Run a shared controller that pauses both the shader loop and the Rive instance on the same offscreen and visibilitychange signals. Keep the hard cap of one Rive instance per page.

---
title: Runtime cost controls and the scroll-to-uniform seam
summary: The controls that keep a background shader within the CI performance budget — clamped device pixel ratio, capped frame rate, pausing off-screen and when hidden, gating mobile to the fallback — and the rule for driving a shader uniform from scroll without adding a motion library for one value.
last_updated: 2026-06-18
applies_to: WebGL2 render loops; Core Web Vitals budgets; the locked motion-system engine ladder
---

# Runtime Cost Controls and the Scroll-to-Uniform Seam

> A background shader runs forever while visible, so its cost is a steady tax on the main thread, the GPU, and the battery. The controls below cap that tax; the seam keeps the scroll integration from pulling in a heavy dependency.

## Contents

- The runtime controls
- Budget accounting against the CI gate
- The render loop with pausing
- The scroll-to-uniform rule
- The driver, concretely

## The runtime controls

| Control | What it does | Why |
|---|---|---|
| Clamp device pixel ratio to about 1.5, cap 2 | Renders a smaller back buffer upscaled by CSS | The cheapest quality-for-speed trade; a background does not need full retina density |
| Cap the frame rate near 30 fps | Skips frames between renders | Gives the GPU time to breathe; a drifting atmosphere does not need 60 fps |
| Pause off-screen | Stops the loop when the canvas leaves the viewport, via IntersectionObserver | Rendering what nobody sees is pure waste; the observer runs off the main thread |
| Pause when hidden | Stops on document.hidden via visibilitychange | Guarantees zero cost and battery drain in a background tab |
| Gate mobile and coarse pointer to the fallback | Shows the static image when the pointer is coarse or the viewport is small | Protects mobile GPUs, batteries, and the interaction metric |

## Budget accounting against the CI gate

The performance budget gate weighs JavaScript bytes in a budget.json and enforces the Core Web Vitals floors. With the default vendored raw WebGL2 helper there is no library to weigh — the cost is the first-party helper plus the shader strings, which is negligible. If the OGL path is chosen instead, measure the tree-shaken minimal subset in the build and write that exact number into the budget line; do not estimate it. The runtime controls above are what keep the loop from regressing the interaction and layout metrics the gate enforces, since the bytes are not the risk here — the per-frame main-thread work is.

## The render loop with pausing

The loop reads a clock and a scroll value, renders, and is started and stopped by the visibility and accessibility logic. It is the single owner of the rAF; nothing else schedules frames.

```js
let rafId = 0;
let running = false;
let lastFrame = 0;
const frameInterval = 1000 / 30; // ~30 fps cap

function startLoop() {
  if (running) return;
  running = true;
  const start = performance.now();
  const tick = (now) => {
    if (!running) return;
    if (now - lastFrame >= frameInterval) {
      lastFrame = now;
      const t = (now - start) / 1000;
      atmosphere.render(t, currentScroll); // currentScroll set by the listener below
    }
    rafId = requestAnimationFrame(tick);
  };
  rafId = requestAnimationFrame(tick);
}

function stopLoop() {
  running = false;
  cancelAnimationFrame(rafId);
}

document.addEventListener('visibilitychange', () => {
  if (document.hidden) stopLoop(); else if (playing) startLoop();
});
```

## The scroll-to-uniform rule

The lightest way to drive a shader uniform from scroll progress is to let the render loop that already runs read a scroll value that a passive listener writes. A scroll listener registered as passive only stores the latest scroll position; the loop reads it each frame and updates the uniform. This adds no library, keeps scrolling on the compositor, and reuses the loop already running.

Do not add a scroll-animation library solely to move one uniform. A general motion library's core is tens of kilobytes gzipped — far more than the whole renderer — and the locked motion-system engine ladder reserves it for scrubbed, pinned, multi-element sequencing that CSS and a plain loop cannot do. The rule: if the page already loads that motion library for other scrubbed animations, you may piggyback on its update callback or a shared timeline to feed the uniform for perfect synchronization. If the page does not already use it, drive the uniform from the renderer's rAF plus a passive scroll listener and add nothing.

## The driver, concretely

```js
let currentScroll = 0;

window.addEventListener('scroll', () => {
  const max = document.documentElement.scrollHeight - window.innerHeight;
  currentScroll = max > 0 ? window.scrollY / max : 0; // 0..1 progress
}, { passive: true });
```

When the page already runs the motion library for scrubbed work, replace the listener with that library's update value mapped to the same 0-to-1 range, and keep the render loop reading currentScroll unchanged.

## Limitations

- The device-pixel-ratio cap, the 30 fps target, and the mobile gating are engineering judgments consistent with platform guidance, not single normative numbers; tune them against measured performance on target hardware.
- Pausing on document.hidden duplicates what most browsers already do by throttling background-tab rAF, but the explicit pause guarantees the behavior and the battery saving rather than relying on the browser.

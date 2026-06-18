---
title: Fallback and accessibility for the shader hero
summary: The no-WebGL fallback choice and detection, WebGL context-loss and restore handling, and the WCAG verdict that a visible pause control is mandatory for a continuously animating decorative background, with the reduced-motion behavior that satisfies the related criteria.
last_updated: 2026-06-18
applies_to: WebGL2; WCAG 2.2 SC 2.2.2 and SC 2.3.3; Astro astro:assets
---

# Fallback and Accessibility for the Shader Hero

> The animation is an enhancement on top of a hero that must work without it — for browsers without WebGL, for a lost GPU context, and for people who need motion to stop. The accessibility requirement here is stronger than it first looks.

## Contents

- The fallback layer
- Detecting WebGL and handling context loss
- The accessibility verdict — a pause control is mandatory
- Reduced motion behavior
- The control, concretely

## The fallback layer

The primary fallback is a static AVIF or WebP image generated through Astro's astro:assets — a representative frozen frame of the atmosphere. It is tiny, art-directable, can serve as the LCP image, and needs no JavaScript. A CSS gradient built from the design tokens is the ultra-light secondary, used for reduced-motion and low-power paths where even an image is more than needed. Avoid a video background — it is the heaviest option and brings its own autoplay, reduced-motion, and bandwidth problems for a purely decorative element. The fallback is not only for missing WebGL; it is the same surface shown under reduced motion and on gated low-power devices, so build it first and treat the shader as the layer on top.

## Detecting WebGL and handling context loss

Detect availability by attempting the context and checking for null; if the helper returns null, show the static fallback and never start a loop. A live context can still be lost — the GPU resets, the tab is backgrounded aggressively, a driver hiccups — so handle the two events. On webglcontextlost, call preventDefault so the context becomes restorable, and cancel the render loop. On webglcontextrestored, recreate every GL resource, because programs, buffers, and textures are all invalid after a loss. If restoration does not arrive, fall back to the static image.

```js
canvas.addEventListener('webglcontextlost', (e) => {
  e.preventDefault();      // makes the context eligible for restore
  cancelAnimationFrame(rafId);
}, false);

canvas.addEventListener('webglcontextrestored', () => {
  rebuildEverything();     // re-link program, re-create buffers, re-set uniforms
  startLoop();
}, false);
```

## The accessibility verdict — a pause control is mandatory

A continuously animating, auto-starting decorative background that runs longer than five seconds and sits in parallel with hero content is a direct target of WCAG SC 2.2.2 Pause, Stop, Hide, a Level A criterion. The criterion requires a mechanism for the user to pause, stop, or hide the moving content unless it is essential, and a decorative hero is not essential. Two points make this non-negotiable:

- prefers-reduced-motion plus aria-hidden do not satisfy SC 2.2.2 on their own. aria-hidden only removes the canvas from the accessibility tree, which addresses screen-reader noise, not the visual-motion requirement. prefers-reduced-motion is an operating-system preference that many users never set, so it cannot be the only mechanism.
- The exemption that the criterion does not apply when the moving content is the only thing on screen does not apply here, because the atmosphere sits behind headline and call-to-action content. Under the whole-page conformance requirement, this Level A gap would taint the entire page.

The conclusion is that a visible, keyboard-operable pause or stop control is effectively mandatory for this component. Provide it, label it, make it reachable by keyboard, and persist the user's choice.

SC 2.3.3 Animation from Interactions, a Level AAA criterion, applies separately to the scroll-driven motion: if scrolling drives shader motion beyond the essential scroll movement, that interaction-triggered animation must be disable-able, which honoring prefers-reduced-motion provides.

## Reduced motion behavior

Under prefers-reduced-motion reduce, do not run the animation loop at all. Show the static fallback entirely, or render a single frozen frame; both are acceptable, and showing the fallback is simplest and guarantees zero motion and zero GPU cost. Do not merely slow the animation — slowing is still motion. Detect the preference in both CSS and JavaScript via matchMedia so the loop never starts, and treat a change of the preference at runtime as a reason to stop and swap to the fallback.

## The control, concretely

```js
const mq = window.matchMedia('(prefers-reduced-motion: reduce)');
let playing = !mq.matches;

const btn = document.querySelector('.atmosphere-toggle'); // visible, keyboard-focusable
btn.setAttribute('aria-pressed', String(!playing));
btn.addEventListener('click', () => {
  playing = !playing;
  btn.setAttribute('aria-pressed', String(!playing));
  try { localStorage.setItem('atmosphere-playing', String(playing)); } catch {}
  if (playing) startLoop(); else stopLoop();
});

// Honor a saved choice and the OS preference on load.
try {
  const saved = localStorage.getItem('atmosphere-playing');
  if (saved !== null) playing = saved === 'true';
} catch {}
if (mq.matches) playing = false; // reduced motion wins on first load
```

## Limitations

- The pause-control requirement is driven by SC 2.2.2 at Level A and is not optional for conformance. SC 2.3.3 is Level AAA — commonly adopted but not required by every legal regime, which typically reference Level AA — so treat the reduced-motion handling for the scroll seam as strongly recommended.
- aria-hidden is correct for a decorative canvas, but it is an accessibility-tree concern only and does nothing for the motion requirement; do not treat it as the accessibility solution.

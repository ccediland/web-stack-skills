---
title: Rive accessibility, reduced motion, and fallback
summary: Rive has no built-in motion accessibility, so this covers the prefers-reduced-motion handling, the WCAG pause-control decision tree, and the static poster fallback for runtime or WASM failure.
last_updated: 2026-06-18
applies_to: @rive-app/canvas@2.38.1, WCAG 2.1 and 2.2
---

# Rive accessibility, reduced motion, and fallback

Rive ships no accessibility handling for motion. Reduced motion, pause controls, and a no-Rive fallback are all the integrator's responsibility. Get this right or the signature moment becomes an accessibility and reliability liability on a perf-gated premium site.

## Contents

- Reduced motion is do-it-yourself
- The WCAG decision tree
- The pause control
- The static poster fallback
- Failure handling

## Reduced motion is do-it-yourself

Read the OS preference at runtime and either start paused or load a reduced artboard. Listen for live changes so toggling the OS setting takes effect without reload.

```ts
const mq = matchMedia('(prefers-reduced-motion: reduce)');
// at init: new Rive({ ..., autoplay: !mq.matches })
mq.addEventListener('change', (e) => {
  if (!rive) return;
  e.matches ? rive.pause() : rive.play();
});
```

If a meaningfully static version exists, the cleaner path is loading an alternative reduced artboard or state machine rather than only pausing, so reduced-motion users still see a coherent still frame.

## The WCAG decision tree

Two distinct criteria apply, and prefers-reduced-motion satisfies only one of them.

| Behavior of the moment | Requirement |
|---|---|
| Autoplays or loops longer than five seconds | A visible, keyboard-operable pause, stop, or hide control for all users (WCAG 2.2.2, Level A) |
| Any non-essential motion from interaction | Honor prefers-reduced-motion (WCAG 2.3.3, Level AAA) |
| Flashing content | No more than three flashes per second (WCAG 2.3.1, Level A) |

prefers-reduced-motion addresses 2.3.3, which is Level AAA, and does not satisfy 2.2.2, which is Level A. So if the moment autoplays or loops past five seconds, a pause control is required for all users regardless of their OS setting. Pausing only on focus does not count as a mechanism to pause under 2.2.2. If the moment is purely interaction-gated, animating only on user hover, click, or scroll and idle otherwise, 2.2.2 is largely satisfied because nothing auto-starts; still honor reduced motion and give any hover-only affordance a keyboard or touch equivalent.

## The pause control

When a control is required, make it a real button, keyboard-operable, with a clear accessible name and pressed state, toggling the Rive instance.

```html
<button id="rive-toggle" type="button" aria-pressed="false">Pause animation</button>
```

```ts
const btn = document.getElementById('rive-toggle') as HTMLButtonElement;
btn.addEventListener('click', () => {
  if (!rive) return;
  const playing = btn.getAttribute('aria-pressed') === 'false';
  playing ? rive.pause() : rive.play();
  btn.setAttribute('aria-pressed', String(playing));
  btn.textContent = playing ? 'Play animation' : 'Pause animation';
});
```

The control must be reachable by keyboard and operable without a pointer. Do not hide it behind hover.

## The static poster fallback

Provide a static image that stands in when Rive cannot run: a WASM or runtime failure, an unsupported environment, or a reduced-motion still. Export a poster frame from the same artboard and render it as the default visible content, so the canvas overlays or replaces it only once Rive loads. Use the image pipeline so it can be AVIF or WebP and can itself be the LCP image.

```astro
---
import { Image } from 'astro:assets';
import poster from './hero-poster.png';
---
<div class="hero-stage">
  <Image src={poster} alt="..." />
  <canvas id="hero-canvas" hidden></canvas>
</div>
```

Reveal the canvas and hide or fade the poster inside the Rive onLoad callback. If init throws, leave the poster in place. Because the Canvas2D runtime uses no WebGL context, there is no webglcontextlost handling to do; that concern belongs to the WebGL background skill.

## Failure handling

Wrap the dynamic import and Rive construction in try and catch, as shown in rive-runtime-setup.md, and on failure do nothing beyond leaving the poster visible. Never block the page or throw to the user. Treat the Rive moment as progressive enhancement on top of a complete static hero.

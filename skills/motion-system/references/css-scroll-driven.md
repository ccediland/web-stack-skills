---
title: CSS scroll-driven animations
summary: The native zero-JS default for reveals and parallax — scroll() and view() timelines, the @supports progressive-enhancement fallback, the Firefox flag and duration quirk, and the precise capability and limit list.
last_updated: 2026-06-17
applies_to: animation-timeline scroll() and view(), Chrome and Edge 115 plus, Safari 26 plus, Firefox behind flag
---

# CSS scroll-driven animations

> The native-first default for reveals, parallax, and progress. Zero JS, zero CSP interaction, runs on the compositor thread. Production-viable but not Baseline, so always ship behind an @supports guard with an accessible default state.

## Contents
- Browser support and Baseline status
- The two timelines: scroll() and view()
- What it can express
- What it cannot express
- The progressive-enhancement fallback
- The Firefox quirks
- prefers-reduced-motion
- Limitations

## Browser support and Baseline status

As of June 2026: Chrome and Edge 115 plus (shipped 2023), Safari 26 plus (added in 26.0), Firefox implemented but behind the layout.css.scroll-driven-animations.enabled flag in stable (default only in Nightly). caniuse reports about 83 percent global usage and the feature is NOT Baseline, because it does not work unflagged in one widely used browser. This is fine for cosmetic enhancement with a clean fallback, not for content whose readability depends on the animation.

Distinguish scroll-driven from scroll-triggered. The continuous-progress animation-timeline family (this document) is broadly available. The discrete animation-trigger property (play a time-based animation once when crossing an offset) is Chrome and Edge only in mid-2026 and is not production-ready; use view() continuous reveals or IntersectionObserver or GSAP for cross-browser discrete triggers.

## The two timelines

scroll() ties animation progress to how far a scroll container has scrolled. Use it for progress bars and effects keyed to overall scroll position.

```css
.progress {
  animation: grow linear forwards;
  animation-timeline: scroll(root block);
}
@keyframes grow { from { transform: scaleX(0); } to { transform: scaleX(1); } }
```

view() ties progress to an element's position as it crosses the viewport, controlled by animation-range with entry, cover, and exit phases. Use it for reveals and per-element parallax.

```css
.reveal {
  animation: fade-up linear both;
  animation-timeline: view();
  animation-range: entry 0% entry 100%;
}
@keyframes fade-up {
  from { opacity: 0; transform: translateY(2rem); }
  to   { opacity: 1; transform: none; }
}
```

## What it can express
Reveals on enter (view), scroll progress (scroll), parallax, sticky and progress bars, staggered reveals via per-element animation-range or animation-delay, and horizontal scroll galleries. All compositor-driven for transform and opacity, so they stay smooth even when the main thread is busy.

## What it cannot express
Scrubbed multi-element cinematic timing with one timeline driving many tweens, true pinning with content lock, snapping to panels, nested timeline orchestration with callbacks, and motion-path animation. Those force GSAP (see gsap-astro.md).

## The progressive-enhancement fallback

Unsupported browsers ignore animation-timeline, so the animation simply does not run. Author the default state as the final visible state, then attach the animation only where supported. This keeps content accessible everywhere.

```css
.reveal { opacity: 1; transform: none; }      /* accessible default */
@supports (animation-timeline: view()) {
  @media (prefers-reduced-motion: no-preference) {
    .reveal {
      animation: fade-up linear both;
      animation-timeline: view();
      animation-range: entry 0% entry 100%;
    }
  }
}
```

Do not use the scroll-timeline JavaScript polyfill: it does not cover the advanced features and it reintroduces the main-thread JS cost the native API removes.

## The Firefox quirks
Firefox requires a non-zero animation-duration for scroll-driven animations to apply; the convention is animation-duration 1ms (the scroll timeline drives progress, not the duration). timeline-scope, animation-range-start, and animation-range-end may still be unsupported in stable. Re-check the layout.css.scroll-driven-animations.enabled flag at author time, since it can move to unflagged at any release.

## prefers-reduced-motion
Gate every scroll-driven animation inside `@media (prefers-reduced-motion: no-preference)`, as shown in the fallback above. The default branch leaves the element in its final visible state, so reduce-motion users get static, readable content. This is gate-relevant for the perf-ci-gates accessibility floor of 0.95.

## Limitations & Out-of-Scope
- Not Baseline; a minority of users (notably Firefox stable) get the static fallback. Acceptable for enhancement, not for animation-dependent content.
- Does NOT do scrub, pin, snap, nested orchestration, or motion paths.
- animation-trigger (discrete) is out of scope until it is cross-browser.

---
name: motion-system
description: Use when building a scroll and interaction animation system for a website — three engines split by job, GSAP for cinematic scroll sequences, native CSS scroll-driven animations for simple reveals, and Motion useAnimate mini for micro-interactions in React islands. Trigger on requests to add scroll animations, pin-and-scrub or parallax or text-split effects, reveal-on-scroll, or island micro-interactions, all behind prefers-reduced-motion. Not for WebGL backgrounds, bespoke state-machine animation, or CSS layout.
---

# motion-system

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

Three engines by job — GSAP (cinematic scroll), native CSS scroll-driven animations (simple reveals), and Motion `useAnimate` mini (micro-interactions in React islands).

## Version pins (seed — re-verify)

- `gsap@3.15.0`, `@gsap/react@2.1.x`
- `motion@12.40.0`

## Outline (author at turn 5)

- One engine per job; `useGSAP` with scope for cleanup; everything behind `matchMedia` and `prefers-reduced-motion`; deferred load per island; CSS scroll-driven as enhancement.
- references — a generic gesture-to-technique matrix (reveal-on-scroll, pin-and-scrub, text-split, parallax, draw-svg, micro-interaction), `useGSAP` scope pattern, reduced-motion wiring.
- Gotchas — the `motion` component is about 34 kb (use `useAnimate` mini, 2.3 kb); Firefox lacks a stable Animation Timeline; SplitText re-split — animate in `onSplit()`; GSAP is client-only; animate transform and opacity, never layout.
- Generic engine split plus a generic gesture matrix — no per-project section list.

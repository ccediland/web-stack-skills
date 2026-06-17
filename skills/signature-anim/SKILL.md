---
name: signature-anim
description: Use when adding one bespoke interactive animated moment to a website — Rive with a state machine driven by named inputs as a public API, mounted lazily in view with an SVG fallback for reduced motion. Trigger on requests to add a Rive animation, an interactive signature animation, or a hero centerpiece, and on choosing between Rive and Lottie or SVG plus GSAP. Not for general scroll animation, WebGL backgrounds, or static illustration.
---

# signature-anim

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

Rive (`@rive-app/canvas`, state machine) for one bespoke interactive animation moment.

## Version pins (seed — re-verify)

- `@rive-app/canvas@2.37.6` (or `@rive-app/webgl2@2.37.8`)
- Not `@rive-app/webgl` (deprecated, frozen at 2.37.0)

## Outline (author at turn 5)

- Deferred mount in view; state-machine input names as the public API; `.riv` as a versioned asset; SVG fallback for reduced motion; `useOffscreenRenderer` for multiple instances.
- references — the Rive mount pattern, state-machine input wiring, fallback, a Rive vs Lottie vs SVG plus GSAP comparison.
- Gotchas — about 200 kb WASM (always defer); concurrent WebGL context limit; `canvas-lite` drops text and audio; serve `.riv` with the correct MIME; reserve dimensions (CLS 0).
- Generic recipe for any signature moment.

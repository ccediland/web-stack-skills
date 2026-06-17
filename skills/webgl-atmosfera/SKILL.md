---
name: webgl-atmosfera
description: Use when building a lightweight animated WebGL background for a website hero — OGL with a custom GLSL shader for sky and light and depth, lazy-loaded after first paint with an image fallback and viewport pausing. Trigger on requests to add an atmospheric or shader background, an animated hero canvas, or a GPU gradient, and on choosing between OGL and Three.js or Pixi or Regl. Not for full 3D scenes, model loading, data visualization, or physics.
---

# webgl-atmosfera

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

OGL with a custom GLSL shader for an atmospheric hero background (sky, light, depth). WebGPU rejected.

## Version pins (seed — re-verify)

- `ogl@1.0.11` (exact pin; stalled dependency — vendor it)

## Outline (author at turn 5)

- Full-screen triangle (no scene graph); lazy after first paint; AVIF fallback; pause off-viewport (IntersectionObserver); cap DPR on mobile.
- references — a generic atmospheric shader, the OGL mount pattern, fallback wiring, an OGL vs Three vs Regl vs Pixi comparison.
- Gotchas — OGL has no recent releases (vendor it); WebGPU is unnecessary and uneven on mobile (iOS WebView lacks it); handle `webglcontextlost`; OffscreenCanvas is not universal; the canvas must not break the CSP style-src.
- Generic background and atmosphere shader for any hero.

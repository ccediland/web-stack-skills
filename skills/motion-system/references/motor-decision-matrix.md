---
title: Motor decision matrix
summary: The native-first escalation ladder across CSS scroll-driven, GSAP plus ScrollTrigger, and Motion, plus the exact CSS-versus-GSAP capability boundary and a when-each table.
last_updated: 2026-06-17
applies_to: astro@6.4.7, gsap@3.15.0, motion@12.40.0
---

# Motor decision matrix

> Choose the lightest engine that does the job. Start at CSS scroll-driven (zero JS), escalate to GSAP only for scrub, pin, snap, nested orchestration, or motion paths, and use Motion only inside a React island that already exists.

## Escalation ladder

1. Try native CSS scroll-driven. It covers reveals, parallax, progress, sticky, staggered reveals, and horizontal galleries at zero JS and zero CSP cost. If it does the job, stop here.
2. If the effect needs scrub, pin with content lock, panel snapping, nested-timeline orchestration, or motion paths, and the page is static, escalate to GSAP plus ScrollTrigger, loaded via a bundled Astro script and lazy-loaded off the critical path.
3. If the motion must live inside a React island that already exists, use Motion Mini useAnimate for imperative sequences, full motion/react for declarative component motion, or the useGSAP hook if you specifically need ScrollTrigger pin or scrub inside that island.

Never add React just to animate. Never ship GSAP for an effect CSS can express. The island already paying for React is the only thing that justifies Motion.

## CSS versus GSAP capability boundary

| Effect | Native CSS scroll-driven | Needs GSAP |
|---|---|---|
| Fade or slide reveal on enter | yes (view()) | no |
| Parallax | yes (scroll() or view()) | no |
| Scroll progress bar | yes (scroll()) | no |
| Sticky or pinned-look header shrink | yes | no |
| Staggered reveals | yes (per-element ranges, animation-delay) | no |
| Horizontal scroll gallery | yes | no |
| Scrubbed multi-element cinematic timing | no | yes |
| True pin with content lock (pin plus scrub) | no | yes |
| Snap to panels | no | yes |
| Nested timeline orchestration with callbacks | no | yes |
| Motion-path animation | no | yes |
| JS-driven values outside the render cycle | no | yes |

The overlap row is reveals and parallax: both engines do them, so the decision rule is native CSS unless you need a row marked needs GSAP.

## When each, in one line

| Situation | Engine |
|---|---|
| Any reveal, parallax, or progress effect | CSS scroll-driven |
| Cinematic scrubbed or pinned section on a static page | GSAP plus ScrollTrigger via bundled script |
| ScrollTrigger pin or scrub needed inside an existing React island | useGSAP in the island |
| Imperative sequence inside an existing React island | Motion Mini useAnimate |
| Declarative component motion already justified by an island | full motion/react |
| Discrete cross-browser trigger (play once on enter) | IntersectionObserver or GSAP, not animation-trigger |

## Why not one engine

GSAP-only would ship about 45 KB gzipped for reveals that cost zero in CSS, and Motion Mini at 2.3 KB beats GSAP for island-local imperative work, so a single engine loses on weight. CSS-only cannot express scrub, pin with content lock, snap, nested orchestration, or motion paths, so it cannot deliver the cinematic moments. The three-engine split maps exactly onto the measured capability boundary: native-first for reveals, scrub-power for cinematic, already-React-cheap for island motion.

## Limitations & Out-of-Scope
- Does NOT cover page or view transitions, WebGL, Rive, Lottie, or UI hover and focus micro-interactions.
- The boundary is drawn for scroll, entrance, and cinematic motion; it does not rank engines for non-scroll animation.

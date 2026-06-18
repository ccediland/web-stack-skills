---
title: Motion inside React islands
summary: When and how to use Motion (formerly framer-motion) only inside a React island that already exists — Mini useAnimate versus full motion/react, verified weights, WAAPI and auto-cleanup, island hydration, and when to prefer it over GSAP.
last_updated: 2026-06-17
applies_to: motion@12.40.0, import motion/react and motion/react-mini, astro@6.4.7
---

# Motion inside React islands

> Use Motion only inside a React island that already exists; never add React to use it. Prefer Mini useAnimate (about 2.3 KB) for imperative sequences, and reserve full motion/react (about 34 KB) for declarative component motion the island already justifies.

## Contents
- The rename
- Mini versus full and weights
- Mini useAnimate
- Full motion/react
- Island hydration
- When Motion over GSAP
- prefers-reduced-motion
- Limitations

## The rename
framer-motion was renamed to Motion. The package is now motion and the React import path is motion/react instead of framer-motion; the API is the same. framer-motion still publishes as an alias but motion is canonical. Current pin motion@12.40.0.

## Mini versus full and weights
Verified gzipped sizes from motion.dev: Mini useAnimate about 2.3 KB (the smallest React animation path, WAAPI-based); the full animate function about 18 KB (timeline sequencing, independent transforms); the full motion component about 34 KB, reducible to about 4.6 KB initial render via LazyMotion plus the m component. For island-local imperative work, Mini at 2.3 KB is dramatically lighter than pulling in GSAP plus ScrollTrigger at about 45 KB.

## Mini useAnimate
Import from motion/react-mini. useAnimate returns a scope ref and an animate function scoped to that ref. Animations started through it are cleaned up automatically when the component unmounts, so there is no manual teardown. It is WAAPI-based and hardware-accelerated.

```jsx
import { useAnimate } from "motion/react-mini";
import { useEffect } from "react";

export default function Pulse() {
  const [scope, animate] = useAnimate();
  useEffect(() => {
    animate(scope.current, { scale: [1, 1.05, 1] }, { duration: 0.6 });
  }, []);
  return <div ref={scope}>...</div>;
}
```

## Full motion/react
Use the declarative component API only when the island already justifies the 34 KB, for example a UI island that needs variants, gestures, layout animations, or AnimatePresence exit animations. Import from motion/react.

```jsx
import { motion } from "motion/react";

export default function Card() {
  return <motion.div initial={{ opacity: 0 }} whileInView={{ opacity: 1 }} />;
}
```

## Island hydration
Motion behaves like any island: Astro server-renders the static markup, and the hook or component runs after hydration. Scroll-into-view and WAAPI features are client-only and safe post-hydration. For motion that should start on scroll, hydrate the island with client:visible so its bundle loads only when the section is near. See csp-and-reduced-motion.md and gsap-astro.md for the hydration and lazy-load patterns.

## When Motion over GSAP
Use Motion when the motion lives inside an existing React island and needs imperative sequencing (Mini) or declarative component motion (full). Use GSAP instead, even inside an island via useGSAP, when you specifically need ScrollTrigger pin, scrub, snap, or nested orchestration that Motion does not cover.

## prefers-reduced-motion
Use the useReducedMotion hook to branch, or wrap the island in MotionConfig reducedMotion user, which disables transform and layout animations while preserving values like opacity. This protects the perf-ci-gates accessibility floor of 0.95.

```jsx
import { useReducedMotion } from "motion/react";

const reduce = useReducedMotion();
// if reduce, render static or opacity-only motion
```

## Limitations & Out-of-Scope
- Only inside a React island that already exists; never a reason to add React.
- Does NOT cover ScrollTrigger pin, scrub, snap, or nested orchestration; use GSAP for those.
- Full motion/react is 34 KB and resists tree-shaking below it; prefer Mini unless the declarative API is needed.

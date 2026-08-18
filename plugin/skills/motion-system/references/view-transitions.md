---
title: View transitions — the same-document ladder, ClientRouter, and cross-document as enhancement
summary: The fourth motion surface — state morphs via the native View Transitions API (Baseline since 2025-10-14) and Motion's animateView, page navigation via Astro's ClientRouter with its automatic a11y, and native cross-document transitions strictly as progressive enhancement with hand-rolled reduced-motion.
last_updated: 2026-08-18
applies_to: astro@7.2.2 (ClientRouter from astro:transitions) · motion@13.1.0 (animateView) · View Transitions API (same-document Baseline Newly Available 2025-10-14)
---

# View transitions

> Three rungs, native-first like every motion decision: the same-document API for in-page state morphs (Baseline — usable today), Astro's ClientRouter for animated NAVIGATION (because it ships the a11y the raw API lacks), and native cross-document transitions only as progressive enhancement (not Baseline, zero Firefox, zero built-in reduced-motion). The trap this reference exists for: the native cross-document one-liner looks free and silently drops every accessibility protection ClientRouter provides.

## Contents

- The support reality
- Rung 1 — same-document morphs (animateView or raw API)
- Rung 2 — navigation with ClientRouter
- Rung 3 — native cross-document as enhancement
- The a11y gap, stated plainly
- Composition with the other engines

## The support reality

| Surface | Support | Baseline |
|---|---|---|
| Same-document (`document.startViewTransition`) | Chrome/Edge 111+, Safari 18+, Firefox 144+ (2025-10-14) | YES — Newly Available since 2025-10-14 |
| Cross-document (`@view-transition` at-rule) | Chrome/Edge 126+, Safari 18.2+, Firefox NO | NOT Baseline |
| View transition TYPES | Firefox 144's initial ship excludes them | partial — feature-detect |

Newly Available means recent browsers only — a same-document transition still needs the graceful path (the DOM change applies unanimated where unsupported, which both the raw API pattern and animateView give for free).

## Rung 1 — same-document morphs

For in-page state changes that deserve a morph (filter toggles, theme-adjacent layout swaps, list reordering) the engine is `animateView` from the `motion` package this catalog already pins — a thin wrapper over the native API that adds spring physics and interruption handling, degrades to the plain DOM update when unsupported, and stays same-document only:

```js
import { animateView } from 'motion';

await animateView(() => {
  container.replaceChildren(...reordered);
});
```

- Zero new dependency: `motion` 13.1.0 is already the in-island engine pin; `animateView` works outside React too.
- Advanced grouping is still rolling out (Safari lacks it) — keep morphs simple.
- Raw `document.startViewTransition(update)` is fine when motion's physics add nothing; wrap in a support check or just call it — unsupported browsers throw only if you assume the return object exists.
- Reduced motion is YOURS to handle on this rung: gate the animated path behind `matchMedia('(prefers-reduced-motion: reduce)')` and run the bare update when reduced — same discipline as every other engine in this skill.

## Rung 2 — navigation with ClientRouter

When the brief wants animated PAGE transitions, the answer on Astro 7 is the built-in `<ClientRouter />` (from `astro:transitions`), not the raw cross-document API — for one decisive reason: it ships the accessibility the native path lacks, automatically.

- ClientRouter injects a CSS media query disabling ALL view-transition animations under `prefers-reduced-motion` — zero config.
- It includes the route announcer (the a11y-deep seam: announcement falls back title → h1 → pathname, so every page keeps a unique title).
- It is NOT deprecated: Astro 7 positions it as the enhanced path while browser APIs mature ("will increasingly become unnecessary" is the docs' own forward-looking note — a review-gate, not a deprecation). The old `<ViewTransitions />` component IS gone (removed in v6); the import is `ClientRouter`.
- Cost consciousness: ClientRouter turns navigation into client-side routing — scripts re-execution semantics, persisted islands, and the announcer come with it. On a mostly-static brochure site with no transition brief, the default remains NO router at all — full page loads are not a defect.
- CSP: ClientRouter is Astro-bundled and hashed automatically; no policy change.

## Rung 3 — native cross-document as enhancement

```css
/* Progressive enhancement ONLY — not Baseline, no Firefox, no built-in a11y */
@media (prefers-reduced-motion: no-preference) {
  @view-transition { navigation: auto; }
}
```

The at-rule buys animated MPA navigation with zero JS in Chromium and Safari 18.2+ — and it is NOT the default because it is not Baseline (Firefox ships nothing) and because the raw feature carries no reduced-motion handling and no announcer. If it ships at all: always inside the reduced-motion media query exactly as above (the query-wrapped at-rule is the entire a11y story), and treat Firefox seeing plain navigation as the designed behavior. Astro provides no config or helper for this rung — it is a stylesheet line, owned here.

## The a11y gap, stated plainly

The native View Transitions surfaces (both documents) ship with NO automatic `prefers-reduced-motion` behavior and NO navigation announcement — verified by absence in the platform docs. ClientRouter provides both. That asymmetry IS the ladder: rung 2 exists because rung 3 is unsafe by default, and any hand-rolled rung-3 usage must reconstruct the reduced-motion gate by hand (the announcer has no hand-rolled equivalent worth building — if announcement matters, use ClientRouter).

## Composition with the other engines

- View transitions are the NAVIGATION/STATE morph surface; scroll-driven CSS, GSAP, and Motion remain the scroll/gesture surfaces — a page can use both, they do not overlap.
- With a11y-deep: the ClientRouter announcer seam (unique titles per page) is already that skill's territory; this reference only points at it.
- With perf-ci-gates: ClientRouter's JS is Astro-internal and small, but it IS a behavior change — audit the composed site after adding it, not before.
- Old `transition:*` directives (`transition:name`, `transition:persist`) belong to ClientRouter pages; the same-document rung uses `view-transition-name` CSS directly.

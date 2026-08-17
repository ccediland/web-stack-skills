---
title: Astro-specific a11y seams and catalog composition boundaries
summary: The ClientRouter route announcer and the per-page title rule, island hydration windows, the dev-toolbar audit posture, and the exact boundaries with motion-system, webgl-atmosfera, signature-anim, and brand-canon-ingest.
last_updated: 2026-08-17
applies_to: astro@7.2.2 (ClientRouter, islands, dev toolbar) · web-stack catalog composition
---

# Astro-specific a11y seams

> Framework mechanics create a11y obligations no generic checklist names. These are the Astro ones, plus the map of which sibling skill owns which shared surface — this skill audits those surfaces, it never re-implements them.

## Contents

- ClientRouter — the route announcer and the title rule
- Islands — the hydration window
- Dev toolbar — triage only
- Composition boundaries
- Contrast and the brand canon

## ClientRouter — the route announcer and the title rule

When a site uses `<ClientRouter />` (client-side navigation), Astro injects an `aria-live="assertive"` announcer on each navigation and announces, in order of availability: the new page's `title`, else its first `h1`, else the pathname.

Hard rules that follow:

- Every page ships a UNIQUE, descriptive `title` — with ClientRouter this is not just SEO hygiene, it is literally what screen-reader users hear on navigation. A shared template title makes every navigation announce the same thing.
- Do not build a custom announcer alongside it (double announcements); do not remove titles trusting the h1 fallback (the fallback exists for failure, not design).
- Under prefers-reduced-motion the router swaps without animation — that complements motion-system's ownership of transition animation; nothing to re-implement here.
- Focus after swap: Astro restores focus where it can; interactive flows that MOVE the user (form submit → confirmation view) should manage focus explicitly (`astro:before-swap` is the hook) — tab-through-after-navigation is a tier-1 smoke line.

## Islands — the hydration window

Islands hydrated with `client:visible` or `client:idle` render interactive-LOOKING markup before their JavaScript attaches: a keyboard user can reach a button during the window where it does nothing. Disciplines:

- Prefer real links and forms (work pre-hydration) over JS-only controls for anything above the fold.
- A control that is inert until hydration should render `disabled` and enable itself on mount — visible truth beats dead interactivity.
- The manual smoke's keyboard pass on a THROTTLED connection is what catches these; CI axe scans post-load DOM and will not.

## Dev toolbar — triage only

Astro's dev toolbar Audit app flags a11y and perf issues per page during `astro dev`. Use it as authoring-time triage; it is explicitly not a replacement for the CI scan or the manual layers (Astro's own docs say as much). Nothing in the three-layer structure delegates to it.

## Composition boundaries

| Surface | Owner | This skill's role |
|---|---|---|
| Reduced-motion per animation engine (CSS scroll-driven, GSAP, Motion) | motion-system | Audit that the posture holds (smoke line); file violations against that skill's recipe |
| WebGL pause/stop control (WCAG 2.2.2-A), canvas fallbacks | webgl-atmosfera | Audit presence + keyboard operability |
| Rive autoplay pause, DIY reduced-motion | signature-anim | Same |
| Lighthouse a11y ≥0.95 floor, workflow file, budgets | perf-ci-gates | Contribute the axe job to the same workflow; never fork the gate |
| Forms (labels, error semantics, consent) | stack-integration-playbook forms recipe | Audit the built form against 3.3.x + announce rules |
| CMS admin app-shell | cms-self-edit | EXCLUDED from the audited set (both gates agree); the client-facing site is the claim surface, the admin is a tool |

## Contrast and the brand canon

On brand-governed sites the palette is not this skill's to change: contrast findings against canon-authored token pairs are filed as OWNER decisions through the brand promotion path (the composition precedent: a CTA pair may pass as large text with the palette intact; hint-role tokens never carry body copy). This skill's job is to MEASURE and NAME the finding with its criterion (1.4.3/1.4.11), attach the failing pair and ratio, and route it — patching token values site-side breaks provenance and is prohibited by the ingest discipline.

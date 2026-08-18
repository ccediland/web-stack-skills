---
title: Component conventions — authoring components against the token layer
summary: The scattered rules gathered in one place (W4 extension) — classes over style attributes under hash CSP, scoped styles, variants by data-attributes, token variables everywhere a design value appears, and the review checklist for component code on this stack.
last_updated: 2026-08-18
applies_to: astro@7.2.2 scoped styles · Tailwind v4 utilities over --ds-* variables · hash-based CSP (web-security-headers) · schemes via data-theme (brand-canon-ingest)
---

# Component conventions

> These rules existed before this file — scattered across the CSP skill's field lessons, the token pipeline, and the composition playbook's seams. Gathered here because component authoring is where they are either honored or violated, one component at a time. None of this is component DESIGN (layout, spacing choices, what a card looks like — that is per-site work); it is the mechanical discipline every component on this stack follows.

## Contents

- The five rules
- Styling surfaces, ranked
- Variants
- Scheme awareness
- What a component may never hardcode
- Review checklist

## The five rules

1. NO style attributes in markup. `style=""` is BLOCKED under the hash-based CSP (hashes cover style elements and bundled scripts, never attributes — the fixture's 84-violation lesson). Every static style goes through classes or scoped styles; only runtime CSSOM writes from JS (GSAP's `element.style.x`) are exempt.
2. Design values come from tokens. Any color, spacing, radius, shadow, duration, or easing a component uses is a `var(--ds-*)` reference (directly or through a Tailwind utility that compiles to one). A hex, a raw px radius, or a magic 300ms in component code is a defect — the value either exists as a token or the token layer grows.
3. Utilities first, scoped styles second. Tailwind utilities over token variables cover most components; a `<style>` block (scoped by default in Astro) earns its place for what utilities express badly — keyframes, complex selectors, `@supports` ladders. Scoped styles compile to bundled CSS and need no CSP thought.
4. Variants are data-attributes, not class soup. A component with modes exposes `data-variant="compact"` / `data-state="open"` and styles against `[data-variant='compact']` — states become inspectable in devtools, reachable from tests, and consistent with how the stack already switches schemes (`data-theme`) and A/B variants (`data-ab-*`).
5. Props type the content, conventions type the look. A component's props carry text, urls, and data; they do NOT carry style values ("color" props invite rule-2 violations). The rare legitimate style prop forwards a token NAME, not a value.

## Styling surfaces, ranked

| Surface | Use for | CSP posture |
|---|---|---|
| Tailwind utilities (over `--ds-*`) | The default — most component styling | Bundled CSS, no interaction |
| Astro scoped `<style>` | Keyframes, complex selectors, @supports guards | Bundled, hashed automatically |
| Global styles (`src/styles/`) | Resets, token bridges, cross-component gates (reveals, A/B) | One file, deliberate |
| `style=""` attribute | NEVER | Blocked by policy |
| JS CSSOM (`el.style.x`) | Runtime animation only (motion-system's engines) | Exempt from style-src |

## Variants

```astro
---
interface Props { variant?: 'default' | 'compact' }
const { variant = 'default' } = Astro.props;
---
<article data-variant={variant} class="card">
  <slot />
</article>

<style>
  .card { padding: var(--ds-semantic-space-surface); }
  .card[data-variant='compact'] { padding: var(--ds-semantic-space-tight); }
</style>
```

The variant union lives in the Props interface (typos fail the editor, not the browser); the attribute is the styling hook; tokens carry the values. Boolean states follow the same shape (`data-state`), and JS toggles attributes — never inline styles — to change them.

## Scheme awareness

Components are scheme-blind by construction: they reference role-level variables (`--ds-scheme-surface`, `--ds-scheme-text-muted`) and the scheme switch (`data-theme` on the root) re-resolves everything above them. A component that "adds dark-mode handling" internally is a smell — if a role is missing for its need, the gap belongs upstream in the token/scheme layer, not in a component-local override. Exact-file brand assets inside components follow brand-canon-ingest's rules (inline verbatim for scheme-reactive SVG pairs — the display-none-twins lesson).

## What a component may never hardcode

- Colors, spacing, radii, shadows, durations, easings (rule 2).
- Font families or weights — the family stack and loaded weights are brand artifacts; components use the inherited stack or a token alias.
- Breakpoints outside the project's scale — utilities carry the scale; a bespoke `@media (min-width: 847px)` needs a written reason.
- Copy that the brand governs — components render slots and props; voice-carrying strings come from content, the dictionary (i18n), or the canon.
- z-index integers — a tiny layer scale (tokens or a documented constant set) beats an arms race of 9999s.

## Review checklist

Before a component merges: zero `style=` attributes (grep is the reviewer) · every design value resolves to a token · variants are data-attributes with typed props · no scheme-conditional logic inside the component · alt/aria obligations met (a11y-deep's floor applies per component, not per page) · animations inside it follow motion-system's reduced-motion gate. Five minutes per component; the CSP violation console and the axe gate catch what review misses, but catching it in review is free.

---
title: Composition recipes
summary: Proven skill subsets for site archetypes. Recipe 1 (brand-heavy site, proven as the Furever integration fixture) is complete; the recipe skeleton at the end is the contract every future archetype recipe fills. A recipe records WHAT composed and WHY, so the next site of the same shape starts from evidence.
last_updated: 2026-08-17
applies_to: catalog v1.0.0
---

# Composition recipes

> A recipe is a proven composition: the subset of skills a site archetype actually needed, the order deltas, the seams it exercised, and the decisions with their evidence. Recipes are appended when a composed build ships its gates green — never written speculatively.

## Contents

- Recipe 1 — Brand-heavy site (proven: Furever integration fixture)
- Recipe skeleton (the contract for future archetypes)
- Pending archetypes

## Recipe 1 — Brand-heavy site (proven: Furever integration fixture)

A site whose visual identity comes from a governed brand repo (interchange contract) and whose content includes a client-edited blog. Proven 2026-08-17 as the Furever integration fixture: 8 skills composed, both CI gates green with active assertions (median scores 1.00 across performance/accessibility/best-practices/seo on all audited pages), deployed live to a Workers Builds branch preview.

### Subset

| Skill | In? | Why |
|---|---|---|
| brand-canon-ingest | YES | Brand repo exists under contract 0.6.0 — it is the only styling source allowed |
| astro-css-tokens | YES | Compiles the vendored token projection (SD v5, usesDtcg, plus the cubic-bezier transform) |
| web-security-headers | YES | Hash-based CSP meta + `_headers` baseline + COOP detach for `/admin/*` |
| perf-ci-gates | YES | Both gates; budgets as resource-summary assertions; admin excluded from the audited set |
| seo-aeo-schema | YES | Typed @graph, canonical/OG head, robots + llms endpoints, sitemap |
| motion-system | YES | CSS scroll-driven reveals + ONE GSAP pin/scrub section, lazy-loaded, reduced-motion via matchMedia |
| webgl-atmosfera | YES (the one visual) | Chosen on evidence, not taste: the canon carries an atmosphere-composition algorithm and the brand repo contains no .riv; fabricating one would violate owner-decides |
| signature-anim | NO | No authored .riv exists; the skill integrates, never authors |
| cms-self-edit | YES | Client-edited blog; Sveltia mounted at public/admin (OAuth Worker = later governance) |

### Composition notes (deltas against the canonical order)

- Ingest ran as a repeatable script (`npm run ingest`): brand gate board ALL-GREEN as a hard precondition inside the script, artifacts vendored (tokens verbatim, schemes serialized role-level with light-default emission order, exact-file logos, five woff2, currentColor icons, favicon, rasterized OG image), and a provenance manifest (brand repo sha + contract pin + sha256 per file). CI never touches the brand repo.
- The shader's palette came from the brand's AUTHORED hex in the scheme spine (the honest numeric source for GL uniforms — the CSS layer stays OKLCH-only), swapped live via a `data-theme` MutationObserver.
- Dark mode is opt-in `data-theme="dark"` with light default and NO `prefers-color-scheme` auto-switch (brand mandate); the no-flash script is single-sourced and manually hashed into the CSP.
- Logo light/dark twins are inlined (raw SVG imports) — hidden twins still download as img elements and trip the offscreen-images audit.
- The one GSAP section loads lazily (IntersectionObserver + dynamic import) and folds to static under reduced motion.
- WebGL ships the two-layer software-GL guard (renderer probe + frame watchdog) — mandatory, not optional: this archetype's shader cost 145 s of TBT on a GPU-less runner before the guard.
- Placeholders are marked VISIBLY when content governance is pending (seed catalog, image bank, privacy notice) — a fixture or staging build never fakes owned content.

### Gates and deploy

- LHCI: 5 runs, mobile preset, floors as errors (performance 0.9, accessibility 0.95, LCP 2500, TBT 300, CLS 0.1), resource budgets as byte assertions, real pages only in `collect.url`.
- Biome: `tailwindDirectives` on; excludes for ingested/generated surfaces (seam 6).
- Deploy: Workers Builds branch preview (assets-only Worker — the site is fully prerendered); verification against the LIVE preview (headers, CSP meta, immutable hashed assets, dark switch, admin reachability).

### What this recipe proves

The full seam catalog (seams 1–9 plus the secondary seams) — this is the maximal composition. A site of this archetype can start from this recipe verbatim and cut what its brief does not need.

## Recipe skeleton (the contract for future archetypes)

Copy this skeleton to add a recipe; every heading is mandatory, one-line answers are fine. A recipe missing gates evidence is a draft, not a recipe.

    ## Recipe N — [archetype name] (proven: [site/fixture, date])

    ### Brief shape
    [What kind of site, what the client edits, what the brand source is, what data it shows.]

    ### Subset
    | Skill | In? | Why |
    [Every catalog skill gets a row — a NO with a reason is as informative as a YES.]

    ### Composition notes (deltas against the canonical order)
    [Only deltas and archetype-specific decisions, each with its evidence.]

    ### Gates and deploy
    [Gate config deltas, budget lines that changed and why, deploy target.]

    ### What this recipe proves
    [Which seams it exercised; what a site of this shape can reuse verbatim.]

## Pending archetypes

Named in the roadmap's client layer (W5), unproven, so no recipe yet: corporate-image site · premium landing · restaurant with self-edited catalog · storytelling site · AI-consultancy site. Each gets its recipe when its first composed build ships gates-green.

## Limitations

- One complete recipe exists; treat it as the maximal case, not the average one.
- Recipes record composition decisions, not per-skill setup — owning skills carry the recipes' implementation depth.

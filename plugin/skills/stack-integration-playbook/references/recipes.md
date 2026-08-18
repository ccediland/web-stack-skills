---
title: Composition recipes
summary: Skill subsets for site archetypes. Recipe 1 (brand-heavy site) is PROVEN by the Furever fixture; recipes 3–5 are DERIVED — every ingredient fixture-proven, the archetype awaiting its first dedicated build. Includes the client-portal frontier (a boundary, deliberately not a recipe) and the recipe skeleton every future archetype fills.
last_updated: 2026-08-18
applies_to: catalog v2 (19 skills)
---

# Composition recipes

> A recipe records WHAT composed and WHY, so the next site of the same shape starts from evidence. Two statuses, stated in every header: PROVEN — a complete build of this archetype shipped its gates green; DERIVED — every ingredient is field-proven (by the fixture or a proven recipe) but no dedicated build of the whole archetype exists yet, so its first build confirms the composition and upgrades the status. Nothing here is speculative: a capability nobody has exercised does not appear in any recipe.

## Contents

- Recipe 1 — Brand-heavy site (PROVEN)
- Recipe 2 — Forms and lead capture (capability recipe, own file)
- Recipe 3 — Lead-gen landing (DERIVED)
- Recipe 4 — Catalog with client self-edit (DERIVED)
- Recipe 5 — Editorial and storytelling site (DERIVED)
- The client-portal frontier (a boundary, not a recipe)
- Pruned archetypes
- Recipe skeleton (the contract)

## Recipe 1 — Brand-heavy site (PROVEN — Furever integration fixture, 2026-08)

A site whose visual identity comes from a governed brand repo (interchange contract) and whose content includes a client-edited blog. Proven as the Furever integration fixture: composed across five waves, FOUR CI gates green with active assertions, deployed live to a Workers Builds branch preview.

### Subset

| Skill | In? | Why |
|---|---|---|
| brand-canon-ingest | YES | Brand repo exists under contract 0.6.0 — it is the only styling source allowed |
| astro-css-tokens | YES | Compiles the vendored token projection (SD v5, usesDtcg, plus the cubic-bezier transform) |
| web-security-headers | YES | Hash-based CSP meta + `_headers` baseline + COOP detach for `/admin/*` |
| perf-ci-gates | YES | The gate workflow; budgets as resource-summary assertions; admin excluded from the audited set |
| seo-aeo-schema | YES | Typed @graph, canonical/OG head, robots + llms endpoints, sitemap |
| motion-system | YES | CSS scroll-driven reveals + ONE GSAP pin/scrub section, lazy-loaded, reduced-motion via matchMedia |
| webgl-atmosfera | YES (the one visual) | Chosen on evidence, not taste: the canon carries an atmosphere-composition algorithm and the brand repo contains no .riv; fabricating one would violate owner-decides |
| signature-anim | NO | No authored .riv exists; the skill integrates, never authors |
| cms-self-edit | YES | Client-edited blog; Sveltia mounted at public/admin (OAuth Worker = later governance) |
| data-layer | YES (added in W1) | Catalog graduated seed→Supabase build-time; webhook→Deploy Hook rebuild cycle |
| i18n-system · media-optimization · a11y-deep | YES (added in W2) | Synthetic second locale; compile image pipeline; the axe gate |
| edge-logic · visual-regression-ci | YES (added in W3/W4) | Client-side A/B; the visual gate with CI-born baselines |
| conversion-patterns | YES (added in W4) | Structural conversion pass found and fixed 3 real leaks |
| auth-simple · client-discovery | Stub / retroactive | Access needs owner action (honest stub); discovery ran backwards as the worked example |

### Composition notes (deltas against the canonical order)

- Ingest ran as a repeatable script (`npm run ingest`): brand gate board ALL-GREEN as a hard precondition inside the script, artifacts vendored (tokens verbatim, schemes serialized role-level with light-default emission order, exact-file logos, five woff2, currentColor icons, favicon, rasterized OG image), and a provenance manifest (brand repo sha + contract pin + sha256 per file). CI never touches the brand repo.
- The shader's palette came from the brand's AUTHORED hex in the scheme spine (the honest numeric source for GL uniforms — the CSS layer stays OKLCH-only), swapped live via a `data-theme` MutationObserver.
- Dark mode is opt-in `data-theme="dark"` with light default and NO `prefers-color-scheme` auto-switch (brand mandate); the no-flash script is single-sourced and manually hashed into the CSP — a pattern that scaled to N pre-paint scripts when the A/B script joined it.
- Logo light/dark twins are inlined (raw SVG imports) — hidden twins still download as img elements and trip the offscreen-images audit.
- The one GSAP section loads lazily (IntersectionObserver + dynamic import) and folds to static under reduced motion.
- WebGL ships the two-layer software-GL guard (renderer probe + frame watchdog) — mandatory, not optional: this archetype's shader cost 145 s of TBT on a GPU-less runner before the guard.
- Placeholders are marked VISIBLY when content governance is pending (seed catalog, image bank, privacy notice) — a fixture or staging build never fakes owned content.

### Gates and deploy

- LHCI: 5 runs, mobile preset, floors as errors (performance 0.9, accessibility 0.95, LCP 2500, TBT 300, CLS 0.1), resource budgets as byte assertions, real pages only in `collect.url`.
- Biome: `tailwindDirectives` on; excludes for ingested/generated surfaces (seam 6). Third gate: axe at zero violations over all built pages. Fourth gate: Playwright screenshots, deterministic via `reducedMotion: 'reduce'`, baselines born in CI.
- Deploy: Workers Builds branch preview (assets + a `main` for the actions route only); verification against the LIVE preview (headers, CSP meta, immutable hashed assets, dark switch, admin reachability).

### What this recipe proves

The full seam catalog — this is the maximal composition. A site of this archetype can start from this recipe verbatim and cut what its brief does not need.

## Recipe 2 — Forms and lead capture (capability recipe, own file)

Lives in `forms-lead-recipe.md` (weight earned its own file): the one-route native architecture — Astro Action on the site's own Worker + Turnstile + Supabase insert-only RLS + native email notification + LFPDPPP consent + the WhatsApp deep-link knob — with per-brief knobs, the gate amendments, and the standing flip conditions that would promote it to a skill. Unlike the archetype recipes, recipe 2 is a CAPABILITY recipe: it composes into any archetype that captures leads.

## Recipe 3 — Lead-gen landing (DERIVED — all ingredients fixture-proven 2026-08; first dedicated build confirms)

### Brief shape

One conversion goal, few pages (a landing, maybe a thanks page and legal), paid or organic traffic arriving with intent, the business closing by form, WhatsApp, or phone. The client edits little or nothing; speed of iteration on the OFFER matters more than content volume. The archetype conversion-patterns exists for.

### Subset

| Skill | In? | Why |
|---|---|---|
| astro-css-tokens | YES | The styling floor; brand repo optional at this size (ingest only if one exists) |
| brand-canon-ingest | Only if a brand repo exists | A landing rarely justifies building a canon first; tokens-only is the honest default |
| web-security-headers | YES | CSP before features, as always |
| perf-ci-gates | YES | Paid traffic makes regressions expensive; the gates are the insurance |
| seo-aeo-schema | YES (trimmed) | Canonical/OG/robots floor; a landing for ads may even noindex — brief decides |
| motion-system | YES (light) | Reveals and hover only; a landing's motion budget is small by design |
| webgl-atmosfera / signature-anim | NO (default) | One-page conversion surfaces rarely earn a visual skill; evidence could flip it |
| cms-self-edit / data-layer / i18n-system | NO (default) | No editors, no catalog, one locale — additions are brief-driven |
| media-optimization | YES if imagery is the pitch | Compile pipeline; the LCP image gets priority |
| a11y-deep | Floor only | The 0.95 gate floor; formal AA only on mandate |
| edge-logic | YES | The client-side A/B is this archetype's iteration engine |
| conversion-patterns | YES — the differentiator | Primary-conversion tree, one-primary-CTA, landing hierarchy, trust slots, the experiment loop |
| visual-regression-ci | YES | One page, two baselines — the cheapest visual gate in the catalog |
| forms + analytics recipes | YES | Capture and measurement; lead_submit server-side, instrumentation BEFORE optimization |

### Composition notes

- Order holds (tokens → security → gates → seo → motion → conversion surfaces); conversion-patterns runs its structural pass BEFORE launch and its experiment loop AFTER analytics has a baseline — instrumentation-first is the recipe's spine, not a suggestion.
- The A/B script joins the no-flash script under the single-source hash pattern (proven: two pre-paint scripts, both hashed, zero CSP drift).
- The primary conversion mirrors how the business closes (form versus WhatsApp versus tel) — the decision tree in conversion-patterns, not taste.
- Trust signals are canon-gated slots; with no canon, they are owner-supplied claims with sources, and empty slots stay EMPTY (marked), never fabricated.

### Gates and deploy

Recipe-1 gate set minus the admin exclusion (no admin); LHCI URL set is tiny (2–3 URLs). Deploy: Workers Builds, assets + actions route.

### What this recipe proves (per-ingredient evidence)

Forms POST 200 on a live deploy · A/B assignment/persistence/flip verified in a real browser · server-side lead_submit env-gated · the conversion pass finding real leaks on a live page · visual gate deterministic under reduced-motion. What no build has confirmed yet: this subset composed WITHOUT the brand/cms/data layers around it — the first dedicated landing build should watch the seams the fixture always had cushioned.

## Recipe 4 — Catalog with client self-edit (DERIVED — all ingredients fixture-proven 2026-08; first dedicated build confirms)

### Brief shape

A business whose site IS its offering list — restaurant menu, service plans, product catalog — where the list lives as business data (rows), prose pages are client-edited (documents), and a contact/lead path closes. The rows-versus-documents boundary is this archetype's first decision, made per content type.

### Subset

| Skill | In? | Why |
|---|---|---|
| astro-css-tokens · web-security-headers · perf-ci-gates · seo-aeo-schema | YES | The foundation four, as always |
| brand-canon-ingest | If a brand repo exists | Same entry-point rule as recipe 1 |
| data-layer | YES — the spine | Catalog as rows: seed JSON day one, Supabase tier 1 when the business updates data itself; webhook→Deploy Hook keeps freshness at ~minutes for zero shipped bytes |
| cms-self-edit | YES | Prose the client edits (about, guides, announcements) — documents, with the content-modeling reference deciding each type's home |
| media-optimization | YES | Catalogs are image-heavy by nature; compile pipeline, constrained layout, priority on the LCP |
| motion-system | YES (light) | Reveals; catalog browsing wants speed, not choreography |
| i18n-system | Brief-driven | Tourist-facing menus flip it in; the multiple_folders + translationKey layout is the proven path |
| a11y-deep | Floor + forms checks | The catalog's filters and forms are where a11y debt accrues |
| edge-logic / conversion-patterns | Later passes | A/B and conversion structure once traffic exists; the appetite is recorded in the brief |
| visual-regression-ci | YES | The catalog page is the money page — it gets a baseline |
| forms + analytics recipes | YES | Lead path and measurement |

### Composition notes

- The FIRST decision is per-content-type homing (rows to data-layer, documents to cms-self-edit) — mis-homing a catalog as markdown is this archetype's classic failure; the boundary table in cms-self-edit's content-modeling reference is the instrument.
- The rebuild cycle (Supabase webhook → Deploy Hook) is what makes "the client updates a price and the site follows in minutes" true WITHOUT shipping a byte of client-side data code — the archetype's whole pitch.
- CI builds from seed (deterministic, zero-network gates); live deploys build from tier 1 — the conscious split recipe 1 proved.
- Prose ABOUT a row (a long dish description) keeps the row's key and lives as a document carrying it — the join happens at build.

### Gates and deploy

Recipe-1 gate set including the admin exclusion (Sveltia is present). LHCI URL set includes the catalog listing AND one item page. Deploy: assets + actions route.

### What this recipe proves (per-ingredient evidence)

The seed→Supabase graduation by content · the webhook cycle end-to-end (~2.5 min) · Sveltia round-trip with schema mirror · compile pipeline at 20 derivatives under a second. Unconfirmed until a dedicated build: a catalog at real scale (hundreds of rows) through tier 1, and filters/search client-side (tier 2's measured 53 KB cost) — both documented in data-layer, neither exercised past synthetic size.

## Recipe 5 — Editorial and storytelling site (DERIVED — all ingredients fixture-proven 2026-08; first dedicated build confirms)

### Brief shape

Content IS the product: long-form guides, cases, a narrative "about", a blog with real cadence. The client (or their writer) edits constantly; reading experience and content structure outrank features. Motion serves the narrative — this is the archetype where the catalog's cinematic range earns its place, and the only one where view transitions are a default consideration rather than an extra.

### Subset

| Skill | In? | Why |
|---|---|---|
| astro-css-tokens · web-security-headers · perf-ci-gates · seo-aeo-schema | YES | Foundation; seo-aeo carries extra weight here (article schema, breadcrumbs, the E-E-A-T surfaces) |
| brand-canon-ingest | If a brand repo exists | Editorial brands usually have one worth governing |
| cms-self-edit | YES — the spine | Collections modeled schema-first (content-modeling reference); the editor experience is a first-class requirement, not an afterthought |
| motion-system | YES (full ladder) | Scroll-driven reveals, ONE pin/scrub set piece if the story earns it, view transitions — ClientRouter when navigation should morph (the only rung with automatic reduced-motion + announcer), same-doc rung otherwise |
| media-optimization | YES | Editorial imagery at compile; posters for any video tiers |
| i18n-system | Brief-driven | The multiple_folders layout was built for exactly this content shape |
| webgl-atmosfera / signature-anim | ONE, only on evidence | The storytelling archetype tempts visuals hardest — the one-visual rule and evidence-gating apply with extra force, not less |
| data-layer / edge-logic | NO (default) | No rows, no experiments — prose sites optimize by editing, not A/B |
| conversion-patterns | Light | Long-form has asks too (subscribe, contact) — the one-primary-CTA rule per page, without the experiment loop |
| a11y-deep | YES, above floor | Reading experiences live and die on semantics, reflow, and focus order; the manual checklist earns its keep here |
| visual-regression-ci | YES | Templates, not every article: baseline the article TEMPLATE and the home |
| analytics recipe | YES | Scroll/read events by data-attribute; the vocabulary extends naturally |

### Composition notes

- Content modeling runs BEFORE any page is built: what is a collection, what is a singleton, what drives rendering as an enum — the schema-first discipline is this archetype's step 2, right after tokens.
- View transitions follow the reference's ladder: no navigation brief → no router; a navigation morph brief → ClientRouter (its automatic a11y is why); cross-document stays a guarded enhancement.
- The reading surface is the perf budget's protagonist: fonts (self-hosted woff2, synthesis off), LCP is the article title or its hero image — never a canvas.
- An empty blog is a deferral, not a scaffold: the archetype ships when real content exists; placeholder articles are marked or the section waits.

### Gates and deploy

Recipe-1 gate set; LHCI URL set covers home + one article of each collection + the heaviest media page. Deploy: assets-only unless forms enter.

### What this recipe proves (per-ingredient evidence)

Localized collections consumed by the glob loader with translationKey · the schema-first round-trip through Sveltia · view-transition rung 1 exercised live (theme morph, spied API call) · the axe gate at zero across all built pages. Unconfirmed until a dedicated build: ClientRouter at full-site scale with the announcer under real navigation, and reading-flow metrics on long pages.

## The client-portal frontier (a boundary, not a recipe)

Customer accounts with per-customer data — order tracking, private memorials, member areas, dashboards — sit OUTSIDE this catalog's archetypes, deliberately:

- The deploy shape changes class: a portal is an APP — every page on-demand behind auth (auth-simple rung 3 territory: Supabase Auth + SSR), per-request data with RLS, an invocation budget, session state. The catalog's premise (prerendered site, build-time data, assets-first Worker) inverts.
- The verdict for a portal item in a brief is out-of-scope WITH THIS ADDRESS: a separate build decision, feasible with pieces the catalog documents (auth-simple's ladder, data-layer's RLS boundary, this playbook's map) but composed under app rules the catalog does not carry.
- The frontier is not forever: if portals recur, they justify their own wave and their own archetype — a decision for the catalog's maintenance cycle, recorded here so the door has a handle.

## Pruned archetypes

- AI-consultancy site — pruned 2026-08-18 with evidence: zero exclusive skills. It is recipe 1 or 3 with an AEO emphasis, and seo-aeo-schema is already foundation in every archetype; an emphasis is not a recipe. If a dedicated build someday shows a genuinely distinct subset, it re-enters through the skeleton like anything else.

## Recipe skeleton (the contract for future archetypes)

Copy this skeleton to add a recipe; every heading is mandatory, one-line answers are fine. State the status honestly: PROVEN (a complete build of the archetype shipped gates-green — name it) or DERIVED (every ingredient proven elsewhere — name where, and name what the first dedicated build must confirm). A recipe missing its evidence lines is a draft, not a recipe.

    ## Recipe N — [archetype name] (PROVEN — [site, date] | DERIVED — [evidence base, date])

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
    [PROVEN — which seams it exercised; what reuses verbatim. DERIVED — the per-ingredient
     evidence map, and the explicit list of what the first dedicated build must confirm.]

## Limitations

- One archetype is PROVEN end-to-end; the three DERIVED recipes are evidence-mapped, not exercised whole — their unconfirmed lists are part of the recipe, read them.
- Recipes record composition decisions, not per-skill setup — owning skills carry the implementation depth.

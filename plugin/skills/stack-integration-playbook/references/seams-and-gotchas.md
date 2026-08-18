---
title: Cross-cutting seams and their field-proven gotchas
summary: The places where two or more skills of the catalog meet and where composed builds actually fail — each seam with its failure mode, the fix, and the owning skill's deep reference. All proven on the first composed build (Furever fixture, 2026-08-17).
last_updated: 2026-08-17
applies_to: astro@7.2.2 · @astrojs/cloudflare@14.2.1 · catalog v1.0.0
---

# Cross-cutting seams and their field-proven gotchas

> A seam is where one skill's output becomes another skill's constraint. Single-skill recipes are individually correct and still compose into broken builds; every entry below happened on a real composed build and none is visible from inside one skill alone.

## Contents

- Seam 1 — CSP × static styling
- Seam 2 — CSP × pre-paint scripts
- Seam 3 — _headers × per-path overrides
- Seam 4 — LHCI gate integrity
- Seam 5 — App-shells × the audited set
- Seam 6 — Biome × generated and ingested surfaces
- Seam 7 — Decorative WebGL × CPU-only environments
- Seam 8 — Ingest provenance × formatters
- Seam 9 — Deploy shape × Workers Static Assets
- Secondary seams (dark scheme, fonts budget, contrast roles, tool hygiene)

## Seam 1 — CSP × static styling (security × tokens, motion, visual)

Astro's hash-based CSP hashes style ELEMENTS and bundled scripts — never style attributes. Every `style="..."` an author writes is rejected silently at runtime while the build stays green: 84 live violations on the fixture's first smoke, zero build warnings. JavaScript CSSOM writes (`el.style.prop = x`, which is how GSAP animates) are exempt — CSP governs markup and resource loading, not the CSSOM API.

Rule for composed builds: all static styling goes through classes (Tailwind arbitrary values cover one-off cases, e.g. `bg-(--ds-scheme-surface)`) or scoped style blocks; per-item variation that tempts an inline style becomes an nth-child rule or a class. Depth: `web-security-headers/references/csp-astro-native.md`.

## Seam 2 — CSP × pre-paint scripts (security × tokens dark mode)

The theme no-flash script must be inline and pre-paint, and Astro does not hash `is:inline` scripts — only bundled ones. Supply its hash manually, and single-source the script string in one module imported by BOTH the Astro config (which hashes it into the policy) and the layout (which renders it). Two copies drift silently: the page keeps working in dev, and the deployed CSP blocks the script with no build error.

## Seam 3 — _headers × per-path overrides (security × cms)

Cloudflare `_headers` rules that match the same path CONCATENATE same-name headers into duplicates — they do not override. Found live: the CMS admin needed a relaxed COOP for its OAuth popup, and the naive `/admin/*` rule produced BOTH values on the response, where browsers honor the stricter one, breaking login. A real override detaches first with a `!` line, then re-declares. Depth: `web-security-headers/references/cloudflare-headers.md`.

## Seam 4 — LHCI gate integrity (perf × everything it gates)

The worst class of composed-build failure: the vacuous gate. LHCI 0.15 refuses `assert.assertions` together with `assert.budgetsFile` — and the treosh action wrapper swallows that crash, so the job goes GREEN having asserted NOTHING. Every skill whose byte cost the gate exists to catch is then unwatched. Express resource budgets as `resource-summary:*` assertions (bytes) inside the one `assertions` block; never pair the two keys.

Standing habit this failure bought: after any CI-config change, open the Lighthouse job log and confirm "Checking assertions against N URL(s)" actually ran, with the N you intended. A green job is a claim, not evidence. Depth: `perf-ci-gates/references/lighthouse-config.md`.

## Seam 5 — App-shells × the audited set (perf × cms)

Pages that are app-shells BY DESIGN (the CMS admin under /admin/ is a 1.9 MB SPA bundle with noindex) tank the Lighthouse floors — LCP ~9 s, seo 0.58 — for reasons that are not defects. Exclude them by listing only real pages in `collect.url`; with `staticDistDir` set, LHCI still serves the whole directory and rewrites each URL's origin to its local server. The admin still deploys and still gets smoke-tested; it just is not held to content-page floors.

## Seam 6 — Biome × generated and ingested surfaces (perf × ingest, tokens)

A composed repo contains three classes of file the linter must not touch, and each fails differently if it does: the Tailwind v4 entry stylesheet needs `css.parser.tailwindDirectives: true` or Biome errors on `@theme`/`@custom-variant`; brand-ingested exact-file SVGs trip the a11y svg-title rule on files that are contractually forbidden to edit; generated token CSS and wrangler's `.wrangler/deploy/config.json` fail format checks on files no human wrote. Excludes are part of the composition, not lint laxity. Depth: `perf-ci-gates/references/biome-setup.md`.

## Seam 7 — Decorative WebGL × CPU-only environments (visual × perf)

On a GPU-less CI runner the "GPU-parallel" fragment shader renders on the CPU via SwiftShader: measured 145 SECONDS of Total Blocking Time (each frame a ~550 ms long task) on a page that is flawless on any developer machine. And the textbook fix is insufficient — `failIfMajorPerformanceCaveat: true` no longer refuses SwiftShader on current Chrome (verified: identical TBT with the flag).

Guard in two layers, both load-bearing: probe `WEBGL_debug_renderer_info` and refuse software renderers by name; and watchdog the first ~10 frames in the render loop — averaging over ~25 ms means kill the canvas permanently and let the static fallback stand. A decorative background is GPU-rendered or absent; there is no acceptable CPU mode. Depth: `webgl-atmosfera/references/renderer-and-library.md`.

## Seam 8 — Ingest provenance × formatters (ingest × perf, everything)

The vendored-ingest model only works if provenance is byte-exact: the manifest records the brand repo's commit plus a sha256 per vendored file, and CI builds standalone. Two silent breaks found on the fixture: the repo formatter reformatted a vendored token JSON after ingest (fix: exclude ingested paths from the formatter — they only change via re-ingest), and the ingest script hashed a compact JSON string while writing an indented one (fix: hash the exact bytes written). Verification is one loop: re-hash every manifest entry against the file on disk; 100% match or the chain is broken.

## Seam 9 — Deploy shape × Workers Static Assets (everything × Cloudflare)

Three facts that only surface when the whole site composes:

- A fully-prerendered site under the Cloudflare adapter leaves `dist/server` EMPTY — the Worker is assets-only, with no `main`. Adding one on-demand route changes the deploy shape; know which side you are on before wiring CI.
- Branch previews on Workers Builds are the native temporary deploy: the branch's `wrangler.jsonc` `build.command` runs inside `versions upload` on push, yielding a stable per-branch alias with no local wrangler auth. Production only moves with an explicit `versions deploy` — a preview can never leak to prod by itself.
- `site` in the Astro config feeds canonical URLs, OG tags, and the sitemap as ABSOLUTE URLs — point it at the stable host per environment, not at a preview alias, or every canonical on the preview lies.

## Secondary seams

- Dark scheme propagation (tokens × everything visual): the scheme switch is `data-theme` on the root; light is the default WITHOUT `prefers-color-scheme` auto-switch when the brand mandates it; emission order of the scheme blocks decides the specificity tie (default first). Anything holding colors OUTSIDE CSS (a shader palette) re-reads on a `data-theme` MutationObserver, sourced from the brand's authored hex — not from computed CSS.
- Logo twins × lazy loading: `display:none` images STILL DOWNLOAD in Chromium even with `loading="lazy"`, and trip the offscreen-images audit. Light/dark logo pairs go inline (import the SVG source raw) — zero requests, swap by class.
- Brand fonts × the byte budget: a real brand's font set (five woff2 ≈ 145 KB on the fixture) nearly fills a default 150 KB font budget by itself — set the font line per brief, not by default.
- Contrast × an intact palette: meet the a11y floor by ROLE assignment, not by recoloring the canon — a 3.9:1 authored pair passes as large text (≥3:1 at 20 px bold); a 3.4:1 "subtle" role is for hints, never body copy.
- Tool hygiene: gitignore a tool's output directory BEFORE its first run (`.lighthouseci/` landed in two fixture commits, once with unusable UNC-named directories); and `astro preview` (workerd) dies if `dist` is rebuilt underneath it — restart it before re-probing or a healthy build looks broken.

## Limitations

- This document indexes CROSS-skill failure modes only; per-skill setup and single-skill gotchas live in each owning skill's references (linked per seam).
- Everything here was proven on one composed build (8 skills, brand-heavy archetype). Archetypes that compose fewer or different skills may not hit every seam; new composed builds append seams here.

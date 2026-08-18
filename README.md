# web-stack-skills

Claude Code skills for building **premium, high-performance websites** on a best-of-all-worlds stack:

**Astro 7** · **Cloudflare Workers Static Assets** · **Tailwind v4 + design tokens** · **GSAP / CSS scroll-driven / Motion** · **raw WebGL2** · **Rive** · **schema-dts / AEO** · **native CSP** · **Lighthouse CI / Biome**

Each skill encodes a vetted verdict, current version pins, a generic recipe, and the gotchas — so you can stand up the same architecture on any project without rediscovering the sharp edges.

## Skills

| Skill | Layer | What it does |
|---|---|---|
| `astro-css-tokens` | foundation | Design tokens (DTCG) to CSS vars + Tailwind v4 `@theme`, no lock-in |
| `web-security-headers` | foundation | Native Astro CSP + Workers `_headers` + SRI |
| `perf-ci-gates` | foundation | Lighthouse CI budgets + Biome gate in GitHub Actions |
| `seo-aeo-schema` | foundation | Typed schema-dts `@graph` + sitemap + `llms.txt` |
| `motion-system` | visuals | GSAP + CSS scroll-driven + Motion, one engine per job |
| `webgl-atmosfera` | visuals | Raw-WebGL2 shader atmosphere for a hero, lazy + fallback |
| `signature-anim` | visuals | Rive state-machine for one bespoke interactive moment |
| `cms-self-edit` | content | Sveltia CMS so a non-technical client self-edits content, git-based, no lock-in |
| `brand-canon-ingest` | brand | Consume a governed brand repo (tokens, schemes, assets, voice) into the site, with provenance |
| `data-layer` | data | Build-time-first external data — collection loaders over a seed or Supabase, one zod contract, rebuild-on-change |
| `i18n-system` | reach | Multi-language with Astro core i18n — default locale at root, typed string dictionary, Sveltia-matched collections, hreflang |
| `media-optimization` | reach | Build-time images with the explicit `compile` service, responsive layouts, priority LCP, and a tiered video ladder |
| `a11y-deep` | reach | WCAG 2.2 AA beyond the Lighthouse floor — axe zero-violations in CI, codified manual smoke, WCAG-EM audits |
| `edge-logic` | edge | A/B, geo, flags, and redirects on Workers — defaulting to LESS at the edge, with the traps that make the obvious approaches wrong |
| `auth-simple` | edge | The least auth that does the job — Worker-level Cloudflare Access for previews and admin, service tokens for CI, Supabase Auth for real portals |
| `visual-regression-ci` | quality | The fourth CI gate — Playwright screenshot comparisons, CI-generated baselines in the repo, SaaS escalation when review UIs earn their place |
| `conversion-patterns` | growth | Structural CRO under measurement — one primary CTA, landing hierarchy, MX trust signals; instrumentation first, never persuasive copy |
| `client-discovery` | client | Turn whatever a client gives you into a feasibility-checked site brief plus a deferral register — interview-first intake, capture by format, per-item verdicts, owner-decided deferrals |
| `stack-integration-playbook` | composition | How the skills compose into ONE site — canonical order, cross-cutting seams, stack map, capability recipes (lead capture, analytics, prefetching, legal template) and archetype recipes |

## Compose per site (the Lego principle)

The bundle is a catalog, not a fixed system: each site composes only the subset its brief needs, and no site takes every skill. Four archetype recipes live in `stack-integration-playbook`:

- **Brand-heavy site** (PROVEN by the integration fixture) — start at `brand-canon-ingest`; the maximal composition.
- **Lead-gen landing** — the foundation four plus `conversion-patterns` as the differentiator, client-side A/B from `edge-logic`, forms and analytics recipes.
- **Catalog with client self-edit** — `data-layer` rows plus `cms-self-edit` documents, with the rows-versus-documents homing decision made per content type.
- **Editorial / storytelling site** — content modeling before pages, the full motion ladder including view transitions, accessibility above the floor.

Customer portals (login, per-customer data) are a documented FRONTIER, not a recipe — the deploy shape changes class. A new skill widens the catalog's reach, never each site's payload; `client-discovery` produces the brief upstream, and the playbook composes from it.

## Proven in composition

The catalog is exercised against a permanent **integration fixture**: a real brand repo consumed by a real 8-skill composed build, with both CI gates green under active assertions (median Lighthouse 1.00 performance / accessibility / best-practices / seo on all audited pages) and a live Workers branch-preview deploy. The cross-cutting lessons that build produced — CSP versus static styles, `_headers` concatenation, vacuous CI gates, software-GL fallbacks, byte-exact ingest provenance — live in the playbook's seams reference, not in folklore.

## Install

This repo is a Claude Code plugin marketplace.

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Skills install at personal or project scope (verified on both). There is no auto-update yet — re-running `install` pulls the latest (git is the source of truth). Only the `plugin/` subdirectory ships to installers — repo docs stay out of your cache. Manual alternative: copy `plugin/skills/<skill>` into `~/.claude/skills/`.

## Versioning

The plugin versions the CATALOG; individual skills carry no version of their own. MAJOR = a breaking catalog change (a skill removed or renamed, a consumed contract broken, the installable layout restructured) · MINOR = a new skill or a versioned extension of an existing one · PATCH = content, descriptions, pins, and fixes with no new surface. Every shipped version is an annotated tag plus a GitHub Release.

## Status

**v2.0.0 — the catalog is COMPLETE.** 19 skills authored, validated, installable, and trigger-tested across five capability waves plus the client layer; the full-catalog roadmap that built it is closed and archived (`archive/`). The catalog now lives in maintenance mode: a drift calendar (17 watches — platform anchors, framework majors, legal reform, pre-GA tools) drives quarterly and on-changelog re-verification, and the permanent integration fixture — a real brand consumed by a real composed build behind FOUR CI gates (Lighthouse, Biome, axe, visual regression) — is the standing test bench for every future re-pin and extension.

## License

MIT © ccediland

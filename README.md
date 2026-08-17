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
| `stack-integration-playbook` | composition | How the skills compose into ONE site — canonical order, cross-cutting seams, stack map, recipes (incl. lead capture) |

## Compose per site (the Lego principle)

The bundle is a catalog, not a fixed system: each site composes only the subset its brief needs, and no site takes every skill.

- **Corporate brochure site** — `astro-css-tokens` + `web-security-headers` + `perf-ci-gates` + `seo-aeo-schema`, plus `motion-system` for reveals. No WebGL, no Rive, no CMS if the content is static.
- **Restaurant site with a self-edited menu** — the same four foundation skills, plus `cms-self-edit` (the owner edits the menu without touching GitHub) and at most ONE visual skill (`webgl-atmosfera` OR `signature-anim`) if the brief justifies it.
- **Brand-heavy site under a governed brand repo** — start at `brand-canon-ingest` and follow composition recipe #1 in `stack-integration-playbook`.

A new skill widens the catalog's reach, never each site's payload. The `stack-integration-playbook` skill is the composition authority: canonical order, the seams where composed builds actually fail, and proven recipes.

## Proven in composition

The catalog is exercised against a permanent **integration fixture**: a real brand repo consumed by a real 8-skill composed build, with both CI gates green under active assertions (median Lighthouse 1.00 performance / accessibility / best-practices / seo on all audited pages) and a live Workers branch-preview deploy. The cross-cutting lessons that build produced — CSP versus static styles, `_headers` concatenation, vacuous CI gates, software-GL fallbacks, byte-exact ingest provenance — live in the playbook's seams reference, not in folklore.

## Install

This repo is a Claude Code plugin marketplace.

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Skills install at personal or project scope (verified on both). There is no auto-update yet — re-running `install` pulls the latest (git is the source of truth). Manual alternative: copy `skills/<skill>` into `~/.claude/skills/`.

## Status

v1.2.0 — the 14 pieces above are authored, validated, installable, and trigger-tested. Wave W2 added `i18n-system`, `media-optimization`, and `a11y-deep`, all exercised on the integration fixture: a synthetic second locale with per-page hreflang (x-default included) on the live preview, a media page built under the explicit `compile` image service, and a THIRD CI gate — an axe zero-violations sweep over every built page. Wave W1 had added `data-layer` (live catalog graduated from seed JSON to Supabase at build time, webhook-triggered rebuilds) and the forms/lead-capture recipe in the playbook. The roadmap in progress (see `execution-plan.md`) continues the capability waves (edge, analytics) toward v2.0.0.

## License

MIT © ccediland

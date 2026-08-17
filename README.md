# web-stack-skills

Claude Code skills for building **premium, high-performance websites** on a best-of-all-worlds stack:

**Astro 6** · **Cloudflare Workers Static Assets** · **Tailwind v4 + design tokens** · **GSAP / CSS scroll-driven / Motion** · **raw WebGL2** · **Rive** · **schema-dts / AEO** · **native CSP** · **Lighthouse CI / Biome**

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

## Compose per site (the Lego principle)

The bundle is a catalog, not a fixed system: each site composes only the subset its brief needs, and no site takes every skill.

- **Corporate brochure site** — `astro-css-tokens` + `web-security-headers` + `perf-ci-gates` + `seo-aeo-schema`, plus `motion-system` for reveals. No WebGL, no Rive, no CMS if the content is static.
- **Restaurant site with a self-edited menu** — the same four foundation skills, plus `cms-self-edit` (the owner edits the menu without touching GitHub) and at most ONE visual skill (`webgl-atmosfera` OR `signature-anim`) if the brief justifies it.

A new skill widens the catalog's reach, never each site's payload.

## Install

This repo is a Claude Code plugin marketplace.

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Skills install at personal or project scope. There is no auto-update yet — re-running `install` pulls the latest (git is the source of truth). Manual alternative: copy `skills/<skill>` into `~/.claude/skills/`.

## Status

All 8 skills are authored and registered. The v1 sprint in progress (see `v1-finalization-plan.md`) migrates the stack to Astro 7, adds a brand-ingestion skill, and validates the bundle against a live reference site. The `stack-integration-playbook` under `deferred/` is a skeleton, intentionally excluded from the installable plugin until it has field substance.

## License

MIT © ccediland

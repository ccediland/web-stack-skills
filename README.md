# web-stack-skills

Claude Code skills for building **premium, high-performance websites** on a best-of-all-worlds stack:

**Astro 6** · **Cloudflare Workers Static Assets** · **Tailwind v4 + design tokens** · **GSAP / CSS scroll-driven / Motion** · **OGL (WebGL)** · **Rive** · **schema-dts / AEO** · **native CSP** · **Lighthouse CI / Biome**

Each skill encodes a vetted verdict, current version pins, a generic recipe, and the gotchas — so you can stand up the same architecture on any project without rediscovering the sharp edges.

## Skills

| Skill | Layer | What it does |
|---|---|---|
| `astro-css-tokens` | foundation | Design tokens (DTCG) to CSS vars + Tailwind v4 `@theme`, no lock-in |
| `web-security-headers` | foundation | Native Astro CSP + Workers `_headers` + SRI |
| `perf-ci-gates` | foundation | Lighthouse CI budgets + Biome gate in GitHub Actions |
| `seo-aeo-schema` | foundation | Typed schema-dts `@graph` + sitemap + `llms.txt` |
| `motion-system` | visuals | GSAP + CSS scroll-driven + Motion, one engine per job |
| `webgl-atmosfera` | visuals | OGL shader background for a hero, lazy + fallback |
| `signature-anim` | visuals | Rive state-machine for one bespoke interactive moment |

## Install

This repo is a Claude Code plugin marketplace.

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Skills install at personal or project scope. There is no auto-update yet — re-running `install` pulls the latest (git is the source of truth). Manual alternative: copy `skills/<skill>` into `~/.claude/skills/`.

## Status

Skills are authored one at a time. The `stack-integration-playbook` under `deferred/` is a skeleton, intentionally excluded from the installable plugin until it has substance.

## License

MIT © ccediland

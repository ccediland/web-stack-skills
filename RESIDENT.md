---
title: web-stack-skills — RESIDENT
updated: 2026-06-16
repo: ccediland/web-stack-skills (public, MIT)
---

# web-stack-skills — RESIDENT

Living doc for this repo — what it is, how it is structured, and where it stands.

## What this is

A public, generic Claude Code **plugin marketplace** shipping 7 skills that encode a premium high-performance website stack. Reusable on any project; one origin project was the first consumer, not the scope. No project-specific content, no secrets, MIT.

## Structure

    .claude-plugin/marketplace.json   marketplace -> one plugin: web-stack
    .claude-plugin/plugin.json        the web-stack plugin manifest
    skills/<name>/SKILL.md            the 7 installable skills
    deferred/stack-integration-playbook/SKILL.md   skeleton, excluded from the plugin
    README.md  LICENSE  RESIDENT.md

Skills live under `skills/` (folder name = skill name). The deferred 8th skill sits under `deferred/` so it is structurally impossible to ship until it has substance.

## The stack (one tool per job)

Astro 6 · Cloudflare Workers Static Assets · Tailwind v4 + Style Dictionary · GSAP + CSS scroll-driven + Motion · OGL · Rive · schema-dts + @astrojs/sitemap + llms.txt · native CSP · Lighthouse CI + Biome.

## Authoring cadence

Each skill is authored through a 5-turn cadence — understanding, pre-research, research, close/decisions, build (validate with skill-creator `quick_validate.py`, package with `package_skill.py`). About one skill per chat.

## Status

- Phase 0 (scaffold) — done.
- 0/7 skills authored (skeletons in place).
- Next — `astro-css-tokens`, turn 1.

## Decisions

- Public + generic + MIT. One marketplace repo, one plugin (`web-stack`) grouping all 7.
- `SKILL.md` = verdict + recipe; `references/` = configs/templates/gotchas (progressive disclosure).
- 8th skill `stack-integration-playbook` deferred as a skeleton; excluded from the installable plugin until it has substance.

## Layout corrections vs the original plan (2026-06-16)

- Skills moved from repo root to `skills/<name>/` (Claude Code plugin convention).
- Added `.claude-plugin/plugin.json` (plugin manifest; the original tree omitted it).
- Deferred 8th skill placed in `deferred/` (outside `skills/`) rather than relying on a marketplace.json exclusion list.

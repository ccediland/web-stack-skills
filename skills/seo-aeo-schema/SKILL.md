---
name: seo-aeo-schema
description: Use when adding structured data and answer-engine optimization to a website — a typed schema-dts @graph with cross-linked @id nodes, @astrojs/sitemap, and llms.txt files for AI citation. Trigger on requests to add JSON-LD, mark up a local business or FAQ or person for E-E-A-T, generate a sitemap, or prepare a site for AI answer engines. Not for keyword research, content strategy, link building, or paid search.
---

# seo-aeo-schema

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

A typed `schema-dts` `@graph` with cross-linked `@id` nodes, `@astrojs/sitemap`, and `llms.txt` / `llms-full.txt`.

## Version pins (seed — re-verify)

- `schema-dts` (Google)
- `@astrojs/sitemap` (Astro 6 compatible)

## Outline (author at turn 5)

- One `@graph` script with cross-referenced `@id`; `areaServed` per region for local businesses; every FAQ question visible in HTML; `Person` for E-E-A-T; `llms.txt` at root plus Markdown of key pages.
- references — a generic LocalBusiness `@graph` example, sitemap config, `llms.txt` template.
- Gotchas — FAQ no longer yields a rich result (Google, 2026-05-07), kept for AI citation not SERP; schema must match visible text; do not inflate AggregateRating; `llms.txt` is voluntary; Google retires FAQ from the Search Console API in 2026-08.
- Generic `@graph` for a local service or business — a sample LocalBusiness, not a specific project.

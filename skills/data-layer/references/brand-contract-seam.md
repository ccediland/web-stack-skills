---
title: The brand-contract seam — data-map, seed file, Supabase, one schema
summary: How this skill implements the brand interchange contract's data-pointer rule — volatile data never frozen in the canon, a data-map naming each dataset's seed file and its Supabase live home under the SAME schema — and the graduation path from seed to live without touching pages.
last_updated: 2026-08-17
applies_to: brand interchange contract 0.6.0 (data pointers) · brand-canon-ingest sites
---

# The brand-contract seam

> When a site consumes a governed brand repo, the brand side already legislated the data layer's shape: volatile data must NOT be frozen into the canon. The canon carries a data-map that POINTS — each dataset to a seed file in the site repo and to its eventual live home in Supabase, under one schema. This skill is the site-side implementation of that rule.

## Contents

- The data-pointer rule
- What lives where
- The graduation path
- Ownership and change protocol
- Placeholder discipline

## The data-pointer rule

The brand canon owns identity (tokens, voice, assets) — never volatile business data (prices, items, availability, schedules). Freezing a price into brand material creates a stale authority that outranks reality. The contract's answer is the data-map: a canon-side index that names each dataset, its schema, its seed location in the consuming site, and its live home. The canon stays true forever; the data stays live somewhere built for it.

## What lives where

| Artifact | Home | Role |
|---|---|---|
| data-map | brand repo (canon) | Names each dataset + where it lives; carries NO values |
| zod schema module | site repo (e.g. `src/data/catalog-schema.ts`) | THE contract artifact both sources conform to (see `build-time-loader.md`) |
| seed file | site repo (e.g. `data/catalog.json`) | Tier-zero source; day-one state; also the honest fixture/staging dataset |
| Supabase table | the project's Supabase | Tier-one live home, SAME schema |
| rebuild wiring | Supabase + Cloudflare | Freshness once tier one is live (see `rebuild-wiring.md`) |

The schema module and the data-map must agree; when the brand repo is vendored at ingest time (brand-canon-ingest), verify the data-map's named datasets against the site's schema modules as part of the ingest review — a dataset the map names that the site does not implement is a visible TODO, not silent drift.

## The graduation path

Day one the site ships on the seed (`file()` loader). Graduation to Supabase is three moves, none of which touch a page:

1. Create the table matching the zod schema; import the seed rows.
2. Swap the collection's loader line from `file()` to the Supabase build-time loader (same schema object).
3. Wire the rebuild cycle and verify it once end-to-end.

Because pages consume `getCollection()` and the schema never changed, the graduation is invisible to templates, CI budgets, and CSP. That invisibility is the point of the seam — and the reason to resist "quick" page-side fetches that would couple templates to a source.

## Ownership and change protocol

- SCHEMA changes are contract changes: they start at the data-map (brand side names the shape), then the zod module, then the table — never table-first with the schema chasing reality.
- VALUE changes are just data: rows in Supabase (or seed edits at tier zero); the rebuild cycle publishes them. No brand governance applies to values — that separation is what the rule buys.
- The site never writes brand-governed content into these tables; editorial content is cms-self-edit territory, and the boundary is worth keeping sharp (rows versus documents).

## Placeholder discipline

A fixture or staging build runs the seed with VISIBLY marked placeholder values (the composition rule from the playbook's recipe #1): synthetic data must announce itself. Graduating placeholders to real values is a data change; graduating the seed to Supabase is a tier change; neither requires the other.

---
title: Governance — projection registration, data pointers, re-ingest
summary: Registering the site in the brand repo's projection registry (machine-checked row format), pointing at volatile data sources, the update protocol, and the promotion path for site-born decisions.
last_updated: 2026-08-17
applies_to: "brand-system-skills 0.6.0"
---

# Governance — projection registration, data pointers, re-ingest

Load when registering the site as a consumer, wiring volatile data, handling a brand change, or promoting a site-side design decision.

## Register the projection — the row is machine-checked

Every consumer of the canon is a row in the brand repo's `satellites/projections.md` registry. Registering the site is part of the ingest, not an optional courtesy: the brand's gates reconcile registered projections against the spine, which is what catches drift (a renamed token, a stale value) on the BRAND side before it silently breaks the site.

The row format (columns: Consumer | Role | consumes | source | Status) is read by the brand's audit gate (R6a):

- `consumes` is machine-readable — token aliases `{tier.category.name}` separated by `;`, or the wholesale form `via tokens/web/` used by shipped web consumers. You MAY pin a value with ` = oklch(...)` in canonical serialized form to assert byte-equality — but every pin is re-verified by the gate, so pin only values you will keep in sync. A stale pin FAILS the brand's board.
- `source` is `derived` for a site (the canon wins; the site derives).
- List only aliases the canon actually defines — a raw value in the registry would create a second source of truth, which the gate exists to prevent.

Worked row (Furever reference site):

```markdown
| `furever-site` (repo `ccediland/furever-site`) | downstream | via `tokens/web/` (astro-css-tokens contract; OKLCH-preserving) + `tokens/schemes/` (C-1 serializer, role-level CSS) | derived | KEEP |
```

Commit the row in the brand repo (its git discipline applies — branch, gates green, merge). The site is not "registered" until that row is on the brand repo's main.

## Data pointers — the site never freezes volatile values

The brand's `satellites/data-map.md` names where each volatile datum lives (typically a catalog JSON in the site repo evolving toward a database, e.g. Supabase, with collections mapping 1:1 to tables). The site reads values from that source at build or runtime:

- Prices, plans, coverage, contact, hours → data source. Never in components, never in content collections as copies.
- The canon defines the MODEL (what a plan is, what fields a folio has); the data source holds the VALUES. Expansion (new region, new plan contents) = add records, no canon edit, no site redeploy logic change.
- When the datastore migrates (JSON → database), the contract holds: same collections, same fields — design the site's data layer against the model, not the storage.

## Update protocol — changes flow root → gates → re-ingest

The brand repo's own discipline: edit the ROOT (spine tokens, canon layers), re-run its derivation tools, end with its gate board green. The site's side of that contract:

- A brand change (color, rule, asset) is made in the brand repo — never patched into the site's copied files. Then re-ingest: re-copy `tokens/web/`, re-run the scheme serializer, re-copy changed assets, rebuild. The ingest steps are idempotent by design.
- Re-ingest triggers: the brand repo's main moved (watch its change log), or the site's decision log says the last ingest is stale. Record the brand repo commit consumed at each ingest — that pin is the site's provenance.
- Never "fix" a brand value site-side even temporarily: the next re-ingest silently reverts it, which is worse than the original bug.

## Promotion path — site-born decisions can become canon

Real design decisions get made on the site (a hover behavior, a compositional rhythm, a selection color). The governance for those:

- Prove it on the site and document it in the site's decision log (its RESIDENT/dev doc) — what was decided, why, where it lives.
- The brand repo's projections satellite declares the promotion path: proven decision → owner review → abstracted to a universal rule → promoted into the canon WITH owner ratification. The site never self-promotes a decision into brand truth.
- Until promoted, the decision is consumer-local: another projection (a deck, a print piece) is not bound by it.

## Limitations

- This document does NOT cover the brand repo's internal gate mechanics (its own docs do) or the site's data-layer implementation (a stack concern, out of this skill).
- Registration requires write access to the brand repo; an external consumer without it hands the row to the brand owner instead.

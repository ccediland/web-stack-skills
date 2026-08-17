---
title: Web-to-stack integration map
summary: The contact points between a composed site and the rest of a native-first stack — data (Supabase), CI (GitHub Actions), deploys and DNS (Cloudflare), secrets (Infisical), workspace and comms — as a map of seams, not tutorials. Depth lives in the owning skill or the wave skill that will own it.
last_updated: 2026-08-17
applies_to: catalog v1.0.0 · Cloudflare Workers Static Assets · stack-canon services
---

# Web-to-stack integration map

> The site is one node in a larger stack. This map names each contact point, the rule that governs it, and where its depth lives (an existing skill, or the roadmap wave that will own it). It is deliberately NOT a set of tutorials — a map entry earns depth only when its owning skill ships.

## Contents

- Data — Supabase and the data-pointer rule
- CI — GitHub Actions
- Deploys, DNS, and previews — Cloudflare
- Secrets — Infisical
- Workspace and accounts — Google Workspace
- Comms, payments, CRM, analytics — future wave homes
- The zero-secret build property

## Data — Supabase and the data-pointer rule

The stack's source-of-truth rule: volatile data (catalog items, prices, availability, schedules) is NEVER frozen into a brand canon or hard-coded into pages. The brand contract expresses this as data pointers — a data-map that names each dataset, its seed file in the site repo (e.g. `data/catalog.json`), and its live home in Supabase under the SAME schema. The site starts on the seed file at build time and graduates to Supabase without a schema change.

Contact points:

- Build-time reads (the default, native-first): page code or a content-collection loader reads the seed JSON — or Supabase directly at build time — and the site stays fully static. Freshness = rebuild.
- Client-side reads (the exception, justified per brief): the browser talks to Supabase directly; this adds the project URL to CSP `connect-src` and puts the client library inside the JS byte budget — both gates must be told.
- Apps versus content: interactive apps backed by Supabase (tracking portals, gated areas) live on their own subdomains and out of the content site's audited set; indexable content stays on the main domain in subdirectories.

Depth: the `data-layer` skill owns loaders, the tier ladder, freshness, and rebuild triggers (shipped W1); the write path is this skill's `forms-lead-recipe.md`.

## CI — GitHub Actions

The repo is the source of truth and Actions is where the catalog's gates live — the two-gate workflow (Lighthouse CI + Biome) from perf-ci-gates runs on push/PR against the real build. Rules at this seam:

- CI builds standalone: no job may need a sibling repo (the vendored-ingest model exists to guarantee this — seam 8 of the seams reference).
- Workflow secrets come from the secrets manager (below) into Actions encrypted secrets; a fully static site needs none.
- Tool outputs are gitignored before the tool's first CI-parity run locally (`.lighthouseci/`).

Depth: `perf-ci-gates` owns the workflow shape.

## Deploys, DNS, and previews — Cloudflare

- Workers Builds is the deploy pipeline: git push → `build.command` from `wrangler.jsonc` → `versions upload`. Branch pushes yield stable per-branch preview aliases on workers.dev (the native temporary deploy — no local wrangler auth); production moves only on an explicit `versions deploy`.
- `public/_headers` carries the static-response header set (web-security-headers owns it); the adapter emits the assets manifest and immutable caching for hashed assets.
- DNS, redirects (www → apex), and the workers.dev/custom-domain split are Cloudflare-zone concerns outside the repo; the repo only assumes a stable canonical host for `site`.
- Edge logic beyond serving (A/B, geo, feature flags) is deliberately absent until its wave ships.

Depth: seam 9 in the seams reference; `edge-logic` (W3) will own Workers logic.

## Secrets — Infisical

Secrets live in the secrets manager, never in git and never in Drive. The composed site's posture:

- A fully static composed site has a ZERO-SECRET build (see below) — this is a property to defend, not an accident.
- Secrets that do exist (CMS OAuth Worker credentials, future form/API keys) flow manager → deployment-platform secret store (Workers secrets, Actions secrets) at configure time; the repo holds names, never values.

## Workspace and accounts — Google Workspace

The business Workspace account (not a personal one) owns the site's third-party registrations: the GitHub OAuth App behind the CMS login, analytics properties, search-console verification. Drive is for documents, never a code or asset home — the repo is the only home for anything the build consumes.

## Comms, payments, CRM, analytics — future wave homes

These stack functions touch the site but have no shipped skill yet; the map marks the seam so composition does not improvise:

| Function | Contact point with the site | Home |
|---|---|---|
| Lead capture / forms | Astro Action on the site's Worker + Turnstile + insert-only storage + notification | RESOLVED — `forms-lead-recipe.md` in this skill (W1) |
| Transactional email | Owner notification via Cloudflare Email Service (verified destination, free); Resend when arbitrary recipients at $0 | `forms-lead-recipe.md` email slot (W1) |
| Analytics / measurement | Script + CSP `connect-src` + event schema (stack canon Cloudflare Web Analytics / PostHog) | `analytics-measurement` (W3) |
| CRM handoff | Lead rows in Supabase are the system of record; a later job syncs to the CRM — the site never talks to the CRM directly | frontier (a forms flip condition) |
| Payments | Off-site or embedded checkout; heavy CSP surface; per-project (Stripe / Mercado Pago) | unassigned — per-project decision |
| WhatsApp / phone comms | Deep links from CTAs, logged as lead events | `forms-lead-recipe.md` knob (W1) |

## The zero-secret build property

A fully composed static site — brand vendored at ingest time, data on seed files, forms not yet wired — builds with ZERO secrets and ZERO network dependencies beyond npm. This property is worth stating because it is the composition's security and reproducibility baseline: CI cannot leak what it does not hold, and any PR that introduces the first secret or the first build-time network call should say so explicitly and name the seam it opens.

## Limitations

- Map, not tutorials: no entry here replaces the owning skill's recipe; wave-owned entries are frontiers until their wave ships.
- Stack-canon service names (Supabase, Resend, PostHog, Infisical) reflect this stack's standing tool choices; swapping a service keeps the seam but changes the depth doc.

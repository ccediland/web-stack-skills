---
title: Client-side escalation — when the browser reads Supabase, and the live-collections boundary
summary: Tier two (client-side supabase-js reads) with its measured byte cost, CSP widening, key posture, and Realtime cap — plus the tier-three boundary where live content collections change the deploy shape. Load only when a brief genuinely needs per-visitor or real-time data.
last_updated: 2026-08-17
applies_to: "@supabase/supabase-js@2.112.3 · astro@7.2.2 live collections · strict hash-based CSP stacks"
---

# Client-side escalation

> Escalating past build-time is buying freshness with bytes, CSP surface, and (at tier three) the deploy shape itself. This reference prices the purchase; the brief decides if it is worth it.

## Contents

- Tier two — client-side reads
- The costs, explicitly
- Key and RLS posture
- Realtime
- Tier three boundary — live collections
- Decision residue

## Tier two — client-side reads

An island (or plain script) creates a supabase-js client with the publishable key and queries directly from the browser. Legitimate cases: availability that changes hourly, per-visitor filtering over datasets too large to prerender every permutation, live pricing. Pattern: keep the client in ONE lazily-loaded module, initialized on first interaction or visibility — never in the critical path of a content page.

## The costs, explicitly

These go in the composition ledger the moment tier two enters a site:

- Bytes: the supabase-js UMD artifact measures 212 KB minified / ~53 KB gzip (measured from the npm package, 2026-08-17); tree-shaken ESM lands somewhat lower but plan with ~50 KB. The CI script-size budget line must grow consciously — on this stack's default 150 KB script budget, the client alone is a third.
- CSP: `connect-src` widens from 'self' to the project origin `https://PROJECT_REF.supabase.co` (add the `wss://` variant only if Realtime is used). This is a deliberate policy edit in the security layer, not a loader detail.
- Key exposure: the publishable key ships in the bundle BY DESIGN; Row Level Security is the actual boundary (next section). This is fine — but it is a posture change from tier one, where no key ever left the build.
- Freshness illusion: client reads are fresh at page-view time only; they do not update a rendered page by themselves (that is Realtime, with its own costs).

## Key and RLS posture

- `sb_publishable_...` only — never a secret key in anything that ships. The key travels on the `apikey` header; never as an Authorization Bearer credential.
- RLS: an explicit read-only policy on exactly the tables/columns the site needs (`for select using (true)` on a public catalog; narrower where rows are not all public). No insert/update/delete policies for the publishable role on read paths — writes belong to the forms recipe's server-side path, not to tier two.
- Assume the key is public knowledge and audit what it can see: `select` policies ARE the privacy boundary.

## Realtime

Subscriptions (`postgres_changes`) make a page update live — at the cost of a websocket, the `wss://` CSP line, and the documented cap: UNAUTHENTICATED Realtime connections are dropped at 24 hours. For a public site that is usually fine (nobody keeps a tab a day), but reconnect logic is on you. Realtime is almost never justified for catalog-style data — a rebuild or a page-view-time read covers it; reserve subscriptions for genuinely live surfaces (order status, availability boards).

## Tier three boundary — live collections

Astro's live content collections (stable since v6; `src/live.config.ts`, `defineLiveCollection`, consumed via `getLiveCollection`/`getLiveEntry`) run their loaders at REQUEST time and REQUIRE on-demand rendering — an adapter-served route. On this stack that flips the Worker from assets-only to assets+main: a deploy-shape change with runtime latency and cost per request. There are no built-in live loaders; you write one.

Treat this as an architecture decision, not a data option: it changes what CI audits, what the CSP delivery looks like on those routes (header, not meta), and what the Worker is. Take it to stack-integration-playbook; the honest trigger is "per-request freshness IS the product" (true dashboards, inventory-critical pages) — not "the client wants fresh-ish data", which tier one's rebuild cycle already serves in minutes.

## Decision residue

Whichever tier a site lands on, record in the project docs: the tier, the reason the lower tier was insufficient, the budget/CSP lines it changed, and the trigger that would move it back down. Escalations without residue become permanent by inertia.

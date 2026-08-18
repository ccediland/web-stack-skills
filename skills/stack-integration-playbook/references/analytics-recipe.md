---
title: Analytics and measurement — the events-layer recipe
summary: The catalog's measurement architecture — Umami events over a cookieless script, the conversion event fired server-side from the existing Astro Action, exact CSP additions, env-gating out of previews, and the aviso paragraph — with the merit map that corrects the stack canon, the GA4 tombstone, per-brief knobs, and the verbatim promotion triggers that would turn this recipe into a skill.
last_updated: 2026-08-17
applies_to: astro@7.2.2 prerendered on Workers Static Assets · Umami Cloud / self-host v3.3.0 · @umami/node@0.4.0 · Cloudflare Web Analytics · new LFPDPPP (DOF 2025-03-20)
---

# Analytics and measurement — the events-layer recipe

> This is a RECIPE, not a skill, by the catalog's own criterion: once the decision table exists, an implementation is ~15 lines of diff — one env-gated script tag, two CSP origin lines, a handful of data attributes, one optional server-side call in the Action, one paragraph in the aviso. The durable asset is the decision content below. The promotion triggers at the end are the standing test; if one fires, this recipe re-scopes as an `analytics-measurement` skill.

## Contents

- The measured evidence
- The merit map (and the canon correction)
- The architecture in one view
- Install — script, CSP, env-gate
- The event vocabulary
- The conversion event, server-side
- Cloudflare Web Analytics — the ambient RUM layer
- The GA4 tombstone
- Consent and the aviso (LFPDPPP)
- Knobs per brief
- Pins and review-gates
- Promotion triggers (verbatim, standing)

## The measured evidence

Per-visitor JS tax of each candidate, measured by direct download + gzip (2026-08-17):

| Script (prod CDN) | gzip |
|---|---|
| Plausible `script.js` | ~1.3 KB |
| Umami `script.js` | ~2.3 KB |
| Cloudflare beacon | ~11 KB |
| PostHog `array.js` | ~78 KB (+ lazy replay recorder) |
| GA4 `gtag/js` | ~143 KB |

GA4 is ~110x Plausible and ~62x Umami — on prerendered pages whose own payload is small, the analytics script can easily be the single heaviest thing on the page. These numbers are the recipe's evidence base; re-measure at each catalog review (the CDN URLs are versionless and unpinnable).

## The merit map (and the canon correction)

1. Umami — WINNER for the lead-gen archetype: custom events at zero marginal JS (attributes), 2.3 KB, real free cloud tier (100K events/mo), cookieless, first-party proxy path to CSP-zero, MIT + Node/Postgres self-host exit (PikaPods).
2. Plausible — co-winner minus the free tier: smallest script, most mature proxy docs; $9/mo floor, and CE self-host drags ClickHouse (2 GB RAM class). The paid-simplicity alternative, not a downgrade.
3. Cloudflare Web Analytics — kept, DEMOTED to ambient RUM layer. It cannot record a conversion (no custom events — its FAQ says "Not yet"), keeps unsampled data 7 days, and samples dashboard queries. Include only when the zone is proxied (auto-inject); its 11 KB manual snippet buys strictly less than Umami's 2.3 KB.
4. PostHog — right tool, DIFFERENT job: product apps with identity, flags, replay (1M free events). On a pure marketing site it is 78 KB plus a `*.posthog.com` wildcard plus `worker-src blob:` to do a 2 KB job.
5. GA4 — tombstone (below).

Canon correction, stated so composed builds stop inheriting the error: "Cloudflare Web Analytics = web-analytics default" is untenable as the MEASUREMENT default — a tool without custom events cannot measure a funnel. Reframed: CF WA = free ambient RUM/CWV layer when the zone is proxied; Umami = the events/funnel default; PostHog unchanged for product apps.

## The architecture in one view

Prerendered pages load the 2.3 KB Umami script (env-gated to production) → soft signals fire as `data-umami-event` attributes on the existing anchors (zero JS written) → the ONE conversion event (`lead_submit`) fires SERVER-SIDE from the Astro Action the forms recipe already ships (adblock-immune) → Supabase remains the system of record; analytics is the funnel diagnostic above it → the aviso de privacidad names the analytics layer; no banner (cookieless, MX).

## Install — script, CSP, env-gate

Layout head, gated so previews and dev never pollute production stats:

```astro
{import.meta.env.PROD && import.meta.env.PUBLIC_UMAMI_ID && (
  <script
    defer
    src="https://cloud.umami.is/script.js"
    data-website-id={import.meta.env.PUBLIC_UMAMI_ID}
  />
)}
```

- The env-gate is TWO conditions: `PROD` (dev builds out) and the site id present (branch previews on Workers Builds build with production mode — keep `PUBLIC_UMAMI_ID` out of preview env so previews stay silent). The id is a public identifier, not a secret — env is for gating, not hiding.
- CSP additions (web-security-headers owns the infrastructure; this is the conscious allowance): `script-src` + `https://cloud.umami.is` and `connect-src` + `https://cloud.umami.is`. External script src needs NO hash — origin allowance only; Umami needs no inline bootstrap.
- First-party proxy variant reaches CSP-zero (script and collect endpoint served same-origin via a Worker route or self-host env vars `TRACKER_SCRIPT_NAME`/`COLLECT_API_ENDPOINT`); remember the verified footgun family — proxies must forward the real visitor IP (`X-Forwarded-For`) or bot filtering silently drops events (documented hard requirement in Plausible's proxy docs; same class of risk applies to any proxied collector).

## The event vocabulary

Standard names, so cross-site reporting stays comparable — attributes on existing markup, zero scripting:

| Event | Where | How |
|---|---|---|
| `lead_submit` | THE conversion | server-side from the Action (below); optionally also client-side on success UI |
| `whatsapp_click` | primary MX contact channel | `data-umami-event="whatsapp_click"` on the wa.me anchor |
| `tel_click` / `mailto_click` | secondary contact intents | same attribute pattern |
| `plan_view` | qualification signal | attribute on the plans CTA, or `umami.track()` from an existing IntersectionObserver if section-reach matters |
| `outbound_click` | maps, social | optional, same pattern |
| `ab_expose` | experiment exposure | fired by the A/B pattern from edge-logic — an experiment without exposure counts is unreadable |

Event names ≤50 chars; extra context via `data-umami-event-*` attributes (stringly). Keep the vocabulary — renaming per site is exactly the drift the promotion triggers watch for.

## The conversion event, server-side

The Action is already the choke point (forms recipe); firing the conversion there makes the one event that pays the bills adblock-immune:

```ts
// inside the lead Action handler, AFTER the Supabase insert succeeds
import umami from '@umami/node';
umami.init({ websiteId: env.UMAMI_ID, hostUrl: 'https://cloud.umami.is' });
ctx.locals.cfContext?.waitUntil?.(
  umami.track('lead_submit', { source: input.source ?? 'form' }).catch(() => {}),
);
```

- Fire-and-forget: a tracking failure must NEVER fail the submission — the lead row in Supabase is the record; the event is diagnostics.
- `@umami/node` 0.4.0 is a tiny fetch wrapper; the events API can also be called with plain `fetch` if a dependency feels heavy for one call.
- Client events cover the soft signals; the server event covers the conversion. Counting both sides of `lead_submit` double-counts — pick server as canonical and treat any client-side copy as UX telemetry, not the KPI.

## Cloudflare Web Analytics — the ambient RUM layer

Include ONLY when the production hostname is on a Cloudflare-proxied zone: enable the zone toggle, the beacon auto-injects at the edge, reports to the site's own `/cdn-cgi/rum` (connect-src `'self'` already covers it; add `script-src https://static.cloudflareinsights.com`). Zero build coupling, free CWV field data — the cheapest RUM in existence. Never paste the manual snippet on non-proxied setups (11 KB + 2 CSP lines for less than Umami provides), and never let the beacon ride on workers.dev previews. It measures traffic and vitals, not the funnel — it coexists with Umami, it does not replace it.

## The GA4 tombstone

GA4 is OFF the catalog's default path, with evidence: 143 KB gzip (the heaviest candidate — ~62x the winner); the most hostile CSP surface of any candidate (googletagmanager + `*.google-analytics.com` + `*.g.doubleclick.net` + `*.google.com` + every `google.tld` enumerated INDIVIDUALLY — CSP wildcards cannot cover TLDs — plus an inline bootstrap to hash, plus dynamically injected sub-scripts, the exact pattern hash CSP exists to resist); data thresholding that withholds report rows at precisely the traffic scale of a small MX business; zero LFPDPPP advantage.

ONE documented exception: the client runs (or will run) Google Ads and needs conversion import / remarketing — Ads-ecosystem gravity, not analytics merit. Then gtag ships as an exception entry (consent posture and aviso updated), never as the default. Search Console needs no GA4 — use GSC directly, unlinked.

## Consent and the aviso (LFPDPPP)

- The law is NEW: LFPDPPP published DOF 2025-03-20, in force 2025-03-21 (INAI dissolved; oversight now Secretaría de Anticorrupción y Buen Gobierno). Tácito consent remains valid for non-sensitive data; the aviso de privacidad remains mandatory.
- Mexico has NO cookie-banner mandate. The cookieless default (Umami / CF WA) ships with NO banner — and the aviso still names the analytics layer, because the disclosure obligation comes from the site's data collection as a whole and the pre-existing Lineamientos required disclosing remote tracking tech. The catalog line, verbatim: cookieless elimina el banner, no el aviso. Never ship "no consent needed" copy.
- Aviso paragraph slot (es-MX, placeholder discipline applies — legal copy is owner/lawyer-gated): a sentence naming the tool, that it runs without cookies and without identifying the visitor, what is measured (páginas visitadas, clics de contacto, país de origen), and the purpose (mejorar el sitio y medir el interés en los servicios).
- If PostHog-with-replay or GA4 ever enters a brief, the posture changes: consent gating (PostHog `cookieless_mode: "on_reject"` / Google consent mode) and an expanded aviso — that is a different, heavier sheet.

## Knobs per brief

| Knob | Default | Alternative and when |
|---|---|---|
| Events home | Umami Cloud free (100K events/mo, 1 website, 6-month retention) | Self-hosted Umami on PikaPods when site #2 exists, volume passes 100K/mo, or retention matters — ONE pod serves unlimited sites. Plausible $9/mo when a client pays for zero-ops EU cloud |
| Origins vs proxy | Direct — two explicit CSP origin lines | First-party proxy (CSP-zero) when measured adblock loss on real traffic justifies it, or the CSP must stay pure-self; forward `X-Forwarded-For` |
| Conversion event | Server-side from the Action + client soft signals | Client-only when the Action's complexity budget is tight — Supabase still counts leads; you lose source attribution on the conversion |
| CF WA | Zone-proxied → auto-inject on; else omit | Never the manual snippet; never on previews |
| Consent UI | None (cookieless) + analytics named in the aviso | Banner/gating only if PostHog-replay or GA4 enters, or EU traffic becomes material |

## Pins and review-gates

| Thing | Pin (2026-08-17) | Review-gate |
|---|---|---|
| Umami self-host | v3.3.0 (2026-08-12, MIT) | active monthly cadence; check breaking notes at majors |
| Plausible CE | v3.2.1 (AGPL; compose drags ClickHouse) | twice-yearly LTS — re-check each half-year |
| `@umami/node` | 0.4.0 | only if the server-side fork is taken; tiny surface |
| `plausible-tracker` npm | 0.3.9 — DO NOT ADOPT (stale vs the live script; use the script tag) | revisit only if Plausible revives it |
| posthog-js | 1.417.4 | near-daily cadence; pinned only if the PostHog fork is taken |
| CDN scripts | versionless — unpinnable | re-measure gzip + re-diff CSP origins at each catalog review; the table above is the baseline |
| CF WA capabilities | no custom events ("Not yet"), 7-day unsampled window, dynamic sampling | if custom events ever land, re-run the merit map — that is the one change that could re-rank CF WA |
| LFPDPPP | new law in force 2025-03-21; secondary regulation still settling | re-check the reglamento status before shipping legal copy |

## Promotion triggers (verbatim, standing)

Promote to SKILL if (a) ≥3 shipped sites show drift in event naming/CSP handling that a loaded convention would prevent, (b) the server-side events fork becomes standard — then every build writes Action code and a skill guiding that code pays for itself, or (c) a consent-UI component enters scope (banner variants, per-tool gating logic) — that's generative, judgment-per-site work, which is skill-shaped.

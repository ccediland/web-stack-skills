---
title: hreflang emission and the detection question
summary: The head-link hreflang recipe with x-default (the seam with seo-aeo-schema), why the sitemap i18n option lost, and the verified table of why automatic language detection is absent on a static Worker — with the one flip that changes it.
last_updated: 2026-08-17
applies_to: astro@7.2.2 · @astrojs/cloudflare@14.2.1 (Workers Static Assets) · Cloudflare Free plan rules
---

# hreflang emission and the detection question

> hreflang is head links, per page, with x-default — generated from the locale map this skill owns and RENDERED by the head component seo-aeo-schema owns. Detection is nothing: on a fully prerendered Worker every server-side mechanism is either inert, unavailable on the Free plan, or billable per hit.

## Contents

- The hreflang recipe
- The seam with seo-aeo-schema
- Why not the sitemap i18n option
- Detection — the verified dead-ends table
- The client-side banner (the ceiling)
- The flip — when a Worker main already exists

## The hreflang recipe

For every page that exists in more than one locale, emit in the head:

```astro
---
import { getAbsoluteLocaleUrl } from 'astro:i18n';
// `alternates` = the locales this page ACTUALLY exists in, resolved by the caller
// (for collection entries: via translationKey — see localized-collections.md)
const { alternates, neutralPath } = Astro.props;
---
{alternates.length > 1 && (
  <>
    {alternates.map((locale) => (
      <link rel="alternate" hreflang={locale === 'es' ? 'es-MX' : locale} href={getAbsoluteLocaleUrl(locale, neutralPath)} />
    ))}
    <link rel="alternate" hreflang="x-default" href={getAbsoluteLocaleUrl('es', neutralPath)} />
  </>
)}
```

Rules the recipe enforces:

- Alternates are emitted ONLY for locales where the page exists — an alternate pointing at a 404 (or at a fallback-rendered page) is worse than none.
- Every alternate set includes the page itself (hreflang is reciprocal and self-referencing) plus `x-default`, pointing at the default locale's URL — the page search engines offer to unmatched languages.
- hreflang values are BCP-47 (`es-MX`, `en`); locale PATH segments (`/en/`) are a routing concern and need not match the hreflang value — `getPathByLocale`/locale `codes` bridge them when they differ.
- Canonicals stay self-referencing per locale page, never cross-locale.

## The seam with seo-aeo-schema

Ownership is bilateral and written in both skills:

- i18n-system owns the MAP — which locales a page exists in, what URL each variant has, what x-default points at. That is the `alternates`/`neutralPath` contract above.
- seo-aeo-schema owns the head SURFACE — its typed head component is the single place link/meta elements are rendered, so the hreflang block is rendered there, fed by this skill's map. One emitter, no duplicates.

If a site composes only one of the two skills: i18n without seo-aeo-schema renders the block in its own layout head (the recipe is self-contained); seo-aeo-schema without i18n emits no hreflang at all (single-locale sites need none).

## Why not the sitemap i18n option

`@astrojs/sitemap` has an `i18n` option that emits `xhtml:link` alternates into the sitemap — a legitimate hreflang channel in general, rejected here on three verified grounds:

- It has NO x-default support.
- It infers locale variants from URL path prefixes; its behavior on partially translated sites is undocumented — the alternates-to-404 risk sits exactly where this stack's sites live (secondary locale added incrementally).
- Google accepts either channel and needs only one; head links give per-page control the sitemap cannot.

Review-gate: if a local build test ever confirms the sitemap option emits alternates only for variants that exist, AND a site is permanently 100% translated, the config-only sitemap route becomes acceptable there. Until tested, head links.

## Detection — the verified dead-ends table

Every "just redirect by browser language" mechanism, checked against this stack:

| Mechanism | Status on a prerendered Worker (Free plan) |
|---|---|
| `Astro.preferredLocale` / `preferredLocaleList` | On-demand-only APIs — inert on prerendered pages |
| Cloudflare Redirect Rules on `accepted_languages` | Impossible — the field exists in Transform Rules ONLY, on any plan |
| Cloudflare Snippets | Not available on Free (quota 0) |
| Bulk Redirects / `_redirects` file | Path-based only (2,000 static + 100 dynamic rules for `_redirects`) — cannot read headers |
| Worker `main` + `run_worker_first` reading Accept-Language | Works, but converts every root hit into a billable invocation (Workers Free = 100k requests/day, hard fail); pure asset requests are free and unlimited |

That table is why the locked posture is NO detection: on a 2-locale site whose default locale is the majority market, the UX gain does not buy a Worker invocation per visit — and auto-redirects by Accept-Language carry SEO risk besides (crawlers mostly present English).

## The client-side banner (the ceiling)

The most this stack ships by default: a dismissible client-side suggestion. Read `navigator.languages`, and if the visitor's language matches a locale the current page is not in, offer the switch — never force it. Persist dismissal in `localStorage`. Zero infrastructure, SEO-safe (crawlers see the page, not a redirect), and honest about its marginal value: most visitors arrive via a language-correct link anyway.

## The flip — when a Worker main already exists

The calculus changes when the site ALREADY ships a `main` script for another reason (the forms recipe's Action endpoint, edge logic from a later wave): the incremental cost of a root redirect drops to near zero. Then, and only then, an Accept-Language 302 at `/` becomes reasonable — with `run_worker_first` scoped to `["/"]` so asset routes stay free, a cookie remembering the visitor's explicit choice (explicit choice ALWAYS beats header detection), and a bot exemption so crawlers index the root as-is. This is a documented knob, not the default; the fixture that proved this wave deliberately did not exercise it.

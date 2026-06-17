---
name: web-security-headers
description: Use when hardening an Astro 6 site deployed to Cloudflare Workers Static Assets — native CSP via Astro's csp option, a _headers file for HSTS and framing and referrer and permissions policies, an .assetsignore to keep docs and secrets out of the asset bundle, plus SRI when a CDN is involved. Trigger on requests to add a Content Security Policy, set security headers, fix style-src or script-src breakage, or pass a security audit. Not for application auth, server-side authorization, or secrets management.
---

# web-security-headers

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

Astro 6 native CSP (`csp: true`, stable) plus a `_headers` file on Workers, an `.assetsignore`, and SRI if a CDN is present.

## Version pins (seed — re-verify)

- `astro@6.4.7`, `@astrojs/cloudflare@13.x`

## Outline (author at turn 5)

- `csp: true` with `strict-dynamic`; self-hosted assets (no external CDN); HSTS preload, COOP, X-Frame DENY, Referrer-Policy, Permissions-Policy.
- references — `_headers` template, `.assetsignore` example, CSP gotcha catalog, SRI plus exact allowlist when a CDN is used.
- Gotchas — Image and Picture inline styles break style-src (issue 14301); unsafe-inline ignored with hashes; Shiki and ClientRouter conflict with CSP; define vars breaks strict CSP; `_headers` does not apply to Worker SSR responses.
- Generic hardening for any Astro + Workers Static Assets site.

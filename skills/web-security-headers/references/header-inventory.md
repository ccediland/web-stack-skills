---
title: Security header inventory
summary: Recommended value, rationale, and caveat for each HTTP security header in the baseline, plus the load-bearing gotchas (HSTS preload, COEP, Cloudflare Managed Transforms).
last_updated: 2026-06-17
applies_to: "Astro 6 on Cloudflare Workers Static Assets"
---

# Security header inventory

Recommended values for a mostly-static site, with the reasoning and the caveat that bites. Set these in public/_headers (static) or middleware (SSR). CSP script-src and style-src are handled by Astro native CSP, not here; this inventory covers the rest plus the CSP frame-ancestors directive.

## Contents
- Header table
- HSTS and preload
- Cross-origin policy trio
- X-Frame-Options versus frame-ancestors
- Cloudflare Managed Transforms

## Header table

| Header | Recommended value | Why and caveat |
|---|---|---|
| Strict-Transport-Security | max-age=63072000; includeSubDomains | Forces HTTPS for two years across subdomains. Omit preload unless you accept a near-permanent commitment (see below). |
| X-Content-Type-Options | nosniff | Stops MIME-type sniffing. No downside; always set. |
| X-Frame-Options | DENY | Blocks framing in legacy browsers. Pair with CSP frame-ancestors for modern ones. Use SAMEORIGIN if you frame your own pages. |
| Referrer-Policy | strict-origin-when-cross-origin | Full referrer same-origin, only the origin cross-origin, nothing on an HTTPS-to-HTTP downgrade. A sane default. |
| Permissions-Policy | geolocation=(), camera=(), microphone=(), payment=(), usb=(), magnetometer=() | Disables powerful features the site does not use. Add back any feature you need, for example camera=(self). |
| Cross-Origin-Opener-Policy | same-origin | Isolates your browsing context from cross-origin openers. Safe for a normal site. |
| Cross-Origin-Embedder-Policy | omit by default | require-corp enables cross-origin isolation but breaks cross-origin embeds unless each sends CORP. Enable only when needed (see below). |
| Cross-Origin-Resource-Policy | same-origin | Limits who can embed your resources. Loosen to cross-origin for assets meant to be embedded elsewhere. |
| Content-Security-Policy | frame-ancestors 'none' | Clickjacking protection the CSP meta element cannot express. Add report-to or sandbox here too if used. |

## HSTS and preload

includeSubDomains plus preload submits the domain to browser preload lists, which is effectively permanent and applies to every subdomain, including ones not yet using HTTPS. OWASP flags the permanent consequences. Ship max-age and includeSubDomains, but do not add preload by default. If you want HSTS managed centrally, enable it zone-wide in the Cloudflare dashboard rather than in _headers, and avoid setting it in both places.

## Cross-origin policy trio

COOP, COEP, and CORP work together for cross-origin isolation. COOP same-origin and CORP same-origin are low-risk defaults. COEP require-corp is the one to leave off: it demands that every cross-origin resource (WebGL textures, Rive files, third-party fonts and scripts) send its own CORP or CORS header, and anything that does not simply fails to load. Turn COEP on only when you have confirmed every cross-origin resource is compliant, for example because you need SharedArrayBuffer.

## X-Frame-Options versus frame-ancestors

X-Frame-Options is the legacy clickjacking control; CSP frame-ancestors is its modern successor and is more expressive. Ship both: X-Frame-Options DENY for older browsers and Content-Security-Policy frame-ancestors 'none' for current ones. They should agree, both denying or both allowing the same origins.

## Cloudflare Managed Transforms

Cloudflare's Add security headers Managed Transform can set several of these headers from the dashboard, but it deliberately omits CSP and Permissions-Policy. If you use it, keep ownership of each header in one place: do not set the same header in both the Managed Transform and _headers, or responses will carry duplicates.

## Limitations

- This document does NOT cover CSP script-src or style-src hashing (see csp-astro-native.md) or the _headers file mechanics (see cloudflare-headers.md).
- Recommended values suit a mostly-static marketing or content site; an app with third-party embeds, payments, or analytics needs per-directive tuning.
- Values current as of 2026-06-17.

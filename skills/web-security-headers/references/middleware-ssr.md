---
title: Headers for server-rendered routes (middleware)
summary: Setting security headers and a real CSP header for on-demand Astro routes via src/middleware.ts, since public/_headers does not apply to Worker responses.
last_updated: 2026-06-17
applies_to: astro@6.4.7
---

# Headers for server-rendered routes

public/_headers only attaches headers to static-asset responses. Any route rendered on demand (prerender set to false, or a fully server output) is served by the Worker, and its response does not pass through _headers. Set those headers in Astro middleware.

For a fully static site this file is not needed. Reach for it only when at least one route is server-rendered.

## Astro middleware

src/middleware.ts runs on every request. Set headers on the response after calling next.

```ts
// src/middleware.ts
import { defineMiddleware } from 'astro:middleware';

export const onRequest = defineMiddleware(async (context, next) => {
  const response = await next();
  const h = response.headers;
  h.set('X-Content-Type-Options', 'nosniff');
  h.set('Referrer-Policy', 'strict-origin-when-cross-origin');
  h.set('X-Frame-Options', 'DENY');
  h.set('Strict-Transport-Security', 'max-age=63072000; includeSubDomains');
  return response;
});
```

This covers on-demand responses with the same baseline you put in _headers for static assets. Keep the two in sync.

## A real CSP header for SSR

On server-rendered pages Astro already emits the computed CSP as a Content-Security-Policy response header automatically when security.csp is enabled, so you do not need middleware for the script and style hashes. Use middleware only to add directives Astro does not generate (for example frame-ancestors), or to switch to report-only during a rollout by setting a Content-Security-Policy-Report-Only header with the same policy.

## Nonces, only if truly required

Astro native CSP is hash-only. If a specific integration genuinely requires a nonce, you have to do it yourself in middleware: generate a random nonce per request, add it to the CSP header you set there, and stamp the same value onto the nonce attribute of the inline tag. Astro will not wire a nonce for you, and this bypasses the native hashing, so treat it as a last resort rather than a default.

## Limitations

- This document does NOT cover static-asset headers (see cloudflare-headers.md) or the native CSP config (see csp-astro-native.md).
- Middleware runs in the Worker; heavy per-request work there has a latency cost. Keep header logic trivial.
- API current as of astro@6.4.7.

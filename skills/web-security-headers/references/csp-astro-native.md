---
title: Astro 6 native CSP (security.csp)
summary: How Astro 6's stable security.csp works — config shape, runtime API, external hashes, delivery model, and incompatibilities.
last_updated: 2026-06-17
applies_to: astro@6.4.7
---

# Astro 6 native CSP

Astro 6 ships a stable Content Security Policy feature under the security.csp config key (it left the experimental flag in 6.0). It is hash-based: at build time Astro computes hashes for every bundled script and style, including ones loaded dynamically, and emits them in the policy. It is not nonce-based.

## Delivery model

Delivery is per page and automatic, decided by whether the page is prerendered:

- Static (prerendered) page: the policy is injected as a meta element with http-equiv set to content-security-policy, inside the head.
- On-demand (server-rendered) page: the policy is sent as a Content-Security-Policy response header.

On Cloudflare with @astrojs/cloudflare 13.7.0 the adapter does not support the staticHeaders adapter feature, so static pages keep the meta element. There is no native path to write Astro's CSP into the _headers file. If you need a real CSP header on a static-hosted Cloudflare project, the only options are a server-rendered route (middleware sets the header) or a third-party build integration that captures headers; neither is the default here.

The meta element cannot carry frame-ancestors, report-uri, report-to, or sandbox, and cannot be report-only. Deliver those from public/_headers as a separate Content-Security-Policy header; the browser enforces the intersection of the meta policy and the header policy.

## Config shape

security.csp accepts true for defaults, or an object:

```js
security: {
  csp: {
    algorithm: 'SHA-256', // or 'SHA-384' or 'SHA-512'
    directives: [
      "default-src 'self'",
      "img-src 'self' data: https://images.cdn.example.com",
      "connect-src 'self'",
      "base-uri 'none'",
      "form-action 'self'",
      "object-src 'none'",
    ],
    styleDirective: {
      resources: ["'self'"],
      hashes: ["sha256-aBcD..."],
    },
    scriptDirective: {
      resources: ["'self'", "https://cdn.example.com"],
      hashes: ["sha256-eFgH..."],
      strictDynamic: false,
    },
  },
}
```

Field notes:

- algorithm sets the hash algorithm for generated hashes. One of SHA-256, SHA-384, SHA-512. Default SHA-256.
- directives is for any directive other than script-src and style-src. Listing script-src or style-src here is a validation error; the validator points you to scriptDirective and styleDirective. These directives apply to every page.
- scriptDirective and styleDirective each take resources (a list of valid sources that overrides Astro's defaults for that directive) and hashes (extra hashes you supply). scriptDirective also takes strictDynamic to enable strict-dynamic.
- resources does not include 'self' by default; add it if you want it kept.
- hashes must start with sha256-, sha384-, or sha512-; other prefixes are a validation error.
- Resources and hashes from config and from the runtime API are merged and deduplicated into the final policy.

## External and CDN resources

External scripts and styles are not covered automatically; Astro only hashes its own bundled output. For a cross-origin script or stylesheet you must either allow its origin in scriptDirective.resources or styleDirective.resources, or supply its hash in the matching hashes list. A hash pins the exact content; an origin allows anything from that host.

## Runtime API (per page)

Set policy from a page or layout frontmatter when a single route needs extra sources:

```astro
---
Astro.csp.insertDirective("default-src 'self'");
Astro.csp.insertScriptResource("https://scripts.cdn.example.com");
Astro.csp.insertStyleResource("https://styles.cdn.example.com");
---
```

These additions merge with the config-level policy and are deduplicated.

## Incompatibilities and constraints

- unsafe-inline is incompatible. Once a hash or nonce is present, modern browsers ignore unsafe-inline, so adding it does nothing and signals a misconfiguration. Do not add it.
- The ClientRouter view-transition component can break under a strict CSP. Test the router path explicitly after enabling CSP.
- Shiki, the default code highlighter, emits inline styles that the policy will reject. Use Prism for code highlighting instead.
- The feature does not run in astro dev because of the Vite dev server. Test with astro build then astro preview.
- Responsive images work out of the box; their styles are computed at build time and hashed automatically, so no extra configuration is needed.

## Limitations

- This document does NOT cover the non-CSP headers (see header-inventory.md) or the _headers file mechanics (see cloudflare-headers.md).
- Nonces are not supported by native CSP; it is hash-only. A nonce path requires a server-rendered route and a real header (see middleware-ssr.md).
- The config shape is current as of astro@6.4.7. Re-verify on upgrade.

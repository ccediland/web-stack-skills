---
title: Subresource Integrity and verification
summary: Adding SRI for cross-origin resources in Astro (manual or astro-shield), and how to verify the deployed security headers.
last_updated: 2026-06-17
applies_to: "astro@6.4.7, @kindspells/astro-shield@1.7.1"
---

# Subresource Integrity and verification

Two loosely related tasks: adding Subresource Integrity to cross-origin resources, and confirming the headers are actually live after deploy.

## Subresource Integrity

Astro has no native SRI. SRI pins the exact bytes of a fetched script or stylesheet via an integrity attribute, so a tampered or swapped CDN file is rejected by the browser. For a mostly same-origin site it adds little, because your own bundled assets are same-origin and already covered by CSP hashes; the value is concentrated in cross-origin resources.

Two ways to add it:

- By hand, for a known cross-origin resource: add integrity with the resource hash and a crossorigin attribute to the tag. Regenerate the hash whenever the resource changes, or it will break.
- With an integration, if there are many such resources: @kindspells/astro-shield can generate SRI hashes during the build. Pin it at 1.3.2 or later (CVE-2024-30250, CVSS 7.5, was patched in 1.3.2; current is 1.7.1). Scope it to SRI only. Its CSP generation overlaps Astro's native CSP, so do not run both as the CSP source; let Astro own CSP and let astro-shield own SRI.

## Verification

CSP cannot be tested in astro dev because of the Vite dev server, so verify against a real build.

- Build and preview: run astro build, then astro preview, or deploy to a preview environment.
- Inspect the response headers with a request inspector, for example curl with the head flag, to confirm each header is present and correct on both a page and a static asset.
- Run a headers scanner such as a public security-headers grading service for a quick external read.
- Run the policy through a CSP evaluator to catch weak or missing directives.
- Open the browser developer console and reload: confirm there are no CSP violation reports from your own scripts or styles, and exercise the view-transition path if the router is in use.

A green grade is not the goal; a policy with no violations on your real pages and no overly broad sources is.

## Limitations

- This document does NOT cover the header set itself (see header-inventory.md) or CSP configuration (see csp-astro-native.md).
- SRI hashes are content-pinned; every change to a pinned cross-origin resource requires a new hash.
- Tool and version references current as of 2026-06-17; re-verify the astro-shield version and advisory status on upgrade.

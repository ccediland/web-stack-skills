---
title: Subresource Integrity and verification
summary: Adding SRI for cross-origin resources in Astro (manual or a small build-time hash script), and how to verify the deployed security headers.
last_updated: 2026-08-17
applies_to: "astro@7.2.2"
---

# Subresource Integrity and verification

Two loosely related tasks: adding Subresource Integrity to cross-origin resources, and confirming the headers are actually live after deploy.

## Subresource Integrity

Astro has no native SRI. SRI pins the exact bytes of a fetched script or stylesheet via an integrity attribute, so a tampered or swapped CDN file is rejected by the browser. For a mostly same-origin site it adds little, because your own bundled assets are same-origin and already covered by CSP hashes; the value is concentrated in cross-origin resources.

Two ways to add it:

- By hand, for one or two known cross-origin resources: add integrity with the resource hash and a crossorigin attribute to the tag. Regenerate the hash whenever the resource changes, or it will break.
- With a small build step, when there are several. @kindspells/astro-shield used to fill this role but is RETIRED from this stack (2026-08-17) — unmaintained since late 2024, community-confirmed inactive, and its peer range stops at astro 4, so Astro 5/6/7 installs only work by ignoring peer deps. Replace it with the zero-dependency recipe below.

Build-time recipe — fetch each pinned cross-origin resource during prebuild, compute its sha384, and render the attribute from the generated map:

```javascript
// scripts/sri-hash.mjs — add "presri" (or fold into prebuild) in package.json
import { createHash } from 'node:crypto';
import { mkdir, writeFile } from 'node:fs/promises';

const RESOURCES = [
  'https://example-cdn.com/widget.min.js',
];

const entries = await Promise.all(RESOURCES.map(async (url) => {
  const buf = Buffer.from(await (await fetch(url)).arrayBuffer());
  const hash = createHash('sha384').update(buf).digest('base64');
  return [url, `sha384-${hash}`];
}));

await mkdir('src/generated', { recursive: true });
await writeFile('src/generated/sri.json', JSON.stringify(Object.fromEntries(entries), null, 2));
```

In the component, import the map and render the tag with `integrity={sri[url]}` plus `crossorigin="anonymous"`. Because the hash is frozen at build time, a swapped CDN file fails closed at runtime until you rebuild; pin an exact resource version in the URL whenever the CDN offers one, so rebuilds do not silently bless new bytes.

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
- Tool and version references current as of 2026-08-17. astro-shield is retired (unmaintained, peer astro ^4); if the project revives with an Astro 7 peer range, re-evaluate against this recipe.

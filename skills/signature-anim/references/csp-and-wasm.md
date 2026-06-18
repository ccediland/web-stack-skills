---
title: Rive CSP and self-hosted WASM in Astro 6
summary: The exact native Astro 6 security.csp additions to run the Rive WASM runtime under a hash-based CSP, how to self-host the WASM and the .riv so everything is same-origin, and the all-in-one variant.
last_updated: 2026-06-18
applies_to: astro@6.4.7, @astrojs/cloudflare@13.7.0, @rive-app/canvas@2.38.1, Node 22
---

# Rive CSP and self-hosted WASM in Astro 6

This is the load-bearing integration. A Rive runtime compiles WebAssembly, and WebAssembly is blocked by default under a Content Security Policy that defines script-src or default-src. By default the runtime also fetches its WASM binary from a third-party CDN. Both must be resolved or the signature animation silently fails to run.

## Contents

- Why the CSP blocks Rive
- The two CSP additions
- Self-hosting the WASM
- Loading the .riv as a same-origin asset
- The all-in-one variant
- What not to add
- Verification

## Why the CSP blocks Rive

A page whose CSP contains script-src or default-src cannot compile or instantiate WebAssembly unless `'wasm-unsafe-eval'` is present in script-src. The browser throws a WebAssembly.CompileError and the animation never starts. The `'wasm-unsafe-eval'` keyword is the narrow, safe grant for this; it permits WASM compilation only and does not enable general JavaScript eval. Source: MDN Content-Security-Policy script-src and W3C CSP Level 3.

Separately, the high-level runtime requests its `rive.wasm` over the network. Unless told otherwise it fetches from a CDN, which would force widening connect-src to that origin and add a cross-origin connection. Self-hosting keeps it same-origin.

The Rive runtime JS chunk itself is bundled by Astro and hashed into script-src automatically, the same way the motion and webgl skills are handled. No manual hash is needed for it.

## The two CSP additions

Express both through Astro 6 native CSP. The `resources` list overrides the default sources for script-src, so `'self'` must be re-listed; Astro's generated per-bundle hashes remain additive and are appended automatically.

```js
// astro.config.mjs
import { defineConfig } from 'astro/config';

export default defineConfig({
  security: {
    csp: {
      directives: ["default-src 'self'", "connect-src 'self'"],
      scriptDirective: {
        resources: ["'self'", "'wasm-unsafe-eval'"],
      },
    },
  },
});
```

The rendered meta directive takes the form `script-src 'self' 'wasm-unsafe-eval' 'sha256-...generatedByAstro'`. The meta-delivered CSP enforces `'wasm-unsafe-eval'` identically to a real HTTP header, because it is a fetch-directive source value, not one of the directives a meta element ignores. This matters on Cloudflare Workers Static Assets, where @astrojs/cloudflare@13.7.0 has no staticHeaders support and the CSP ships as a meta element; the WASM keyword still applies. No public/_headers entry is required for the Rive CSP.

The meta element ignores only frame-ancestors, sandbox, and the reporting directives. Rive needs none of those, so there is no fallback to a real header for this skill. A real header would only be needed if you required one of those ignored directives, in which case you would make the page on-demand so the adapter emits a Content-Security-Policy header, or add the one directive via a Cloudflare rule.

## Self-hosting the WASM

Import the WASM with the explicit `?url` suffix so Vite treats it as a static asset and returns its hashed same-origin URL, then point the RuntimeLoader at it before constructing any Rive instance.

```ts
import wasmUrl from '@rive-app/canvas/rive.wasm?url';
import { Rive, RuntimeLoader } from '@rive-app/canvas';

RuntimeLoader.setWasmUrl(wasmUrl);
```

The `?url` suffix is required. A bare `.wasm` import is ambiguous under Vite: the native `?init` form returns an instantiation function, not a URL, and a plain import can be inlined as base64 below the asset inline limit. For Rive you want a URL string. Vite has built-in `.wasm` asset handling, so no extra loader or assetsInclude entry is needed for this URL path. Astro 6 runs Vite with Rolldown; verify the `?url` import compiles at build, as Rolldown asset behavior can differ.

The self-hosted WASM lands in dist/client and is served same-origin, satisfied by connect-src 'self'. Long-cache it: emit `Cache-Control: public, max-age=31557600, immutable` for the hashed asset path so the heavy binary is fetched once.

## Loading the .riv as a same-origin asset

Prefer a `?url` import so the file is content-hashed into dist/client and is unambiguously same-origin.

```ts
import rivUrl from './hero.riv?url';
// const riv = new Rive({ src: rivUrl, canvas, stateMachines: 'Hero', ... });
```

Use public/ only when the `.riv` must be swapped without a rebuild; it is served verbatim at its path with no hashing. Either way it is same-origin under connect-src 'self'. Avoid a non-root Astro `base`; a known adapter bug can emit assets without the base prefix so the Workers asset binding returns 404.

## The all-in-one variant

`@rive-app/canvas-single` inlines the WASM into the JS bundle, so there is no separate WASM network request and no connect-src entry is needed for the WASM. It still compiles WebAssembly, so `'wasm-unsafe-eval'` is still required. The trade is a much larger single JS file that caches as one unit. Prefer the non-inlined `@rive-app/canvas` so the large WASM is a separately cacheable resource; reserve canvas-single for a hard requirement to eliminate the second request. There is no canvas-single-lite.

## What not to add

- Do not use `'unsafe-eval'`. Rive does not need it for WASM, and per the spec it overrides any `'wasm-unsafe-eval'` in the policy, weakening it.
- Do not add worker-src or blob:. These are only touched by OffscreenCanvas worker scenarios, which a single-instance Canvas2D design does not use.
- Do not widen connect-src to a CDN. Self-host instead.

## Verification

CSP is not enforced in astro dev. Run astro build then astro preview, confirm the page meta CSP contains `'wasm-unsafe-eval'` plus the Astro hashes, and confirm the Rive canvas renders with no CSP violation in the console. Measure the served WASM with a request that sends Accept-Encoding br or gzip and read Content-Length, then set the perf budget against that number rather than assuming a small runtime.

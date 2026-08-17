---
title: Locale routing, page structure, and the language switcher
summary: The exact astro.config i18n block for default-at-root routing, the src/pages layout per locale, the astro:i18n URL helpers, the switcher pattern, fallback semantics, and localized 404 notes.
last_updated: 2026-08-17
applies_to: astro@7.2.2 core i18n (no integration) · @astrojs/cloudflare@14.2.1
---

# Locale routing, page structure, and the language switcher

> Routing is config plus folder layout — nothing else. The default locale renders at the root, each secondary locale under its path prefix, and every URL the site emits for another locale comes from an `astro:i18n` helper, never from string concatenation.

## Contents

- The config block
- Page structure on disk
- URL helpers
- The language switcher
- Fallback semantics
- Localized 404 and edge notes

## The config block

```js
// astro.config.mjs
export default defineConfig({
  site: 'https://example.com', // required for absolute locale URLs and hreflang
  i18n: {
    locales: ['es', 'en'],
    defaultLocale: 'es',
    routing: {
      prefixDefaultLocale: false, // es at root, /en/... prefixed — the locked default
    },
  },
});
```

- `prefixDefaultLocale: false` is Astro's own default; stating it explicitly documents the decision. A site that adds a language this way keeps every existing URL.
- `redirectToDefaultLocale` only applies when `prefixDefaultLocale: true` (it controls the root `/` redirect). With the locked shape there is no root redirect to configure at all — that is part of why the shape was chosen.
- `routing: "manual"` disables Astro's i18n middleware entirely and unlocks the low-level helpers (`redirectToDefaultLocale`, `notFound`, `middleware()`); it is for on-demand architectures and out of scope here.
- Locales can be objects (`{ path: 'en', codes: ['en', 'en-US'] }`) when one path segment must answer for several BCP-47 codes; plain strings suffice for most briefs.

## Page structure on disk

With `prefixDefaultLocale: false`, the default locale's pages live at the top of `src/pages` and each secondary locale gets a folder named exactly like its locale path:

    src/pages/
      index.astro          -> /            (es)
      contacto.astro       -> /contacto/   (es)
      en/
        index.astro        -> /en/         (en)

Astro does NOT generate secondary-locale pages for you (outside of `fallback`, below): a page exists in a locale because a file exists for it. A page with no translation simply has no file — and gets no hreflang block (see `hreflang-and-detection.md`).

## URL helpers

All from `astro:i18n`; the absolute variants require `site` to be set.

| Helper | Use |
|---|---|
| `getRelativeLocaleUrl(locale, path?)` | Every internal link that crosses or names a locale |
| `getAbsoluteLocaleUrl(locale, path?)` | Canonical/OG URLs for a specific locale |
| `getRelativeLocaleUrlList(path?)` / `getAbsoluteLocaleUrlList(path?)` | One URL per configured locale for the SAME path — the switcher and hreflang input |
| `getPathByLocale` / `getLocaleByPath` | Mapping between locale codes and path segments when they differ |
| `Astro.currentLocale` | The locale of the page being rendered — works on prerendered pages |

Never build locale URLs by hand (`'/' + lang + path`): the helpers honor `prefixDefaultLocale`, trailing-slash config, and locale-path mapping in one place.

## The language switcher

For `.astro` pages the switcher is the helper applied to the current path:

```astro
---
import { getRelativeLocaleUrl } from 'astro:i18n';
const current = Astro.currentLocale;
// strip the locale prefix to get the locale-neutral path of this page
const neutralPath = Astro.url.pathname.replace(/^\/en(\/|$)/, '/');
---
<nav aria-label="Language">
  <a href={getRelativeLocaleUrl('es', neutralPath)} aria-current={current === 'es' ? 'true' : undefined}>ES</a>
  <a href={getRelativeLocaleUrl('en', neutralPath)} aria-current={current === 'en' ? 'true' : undefined}>EN</a>
</nav>
```

For collection entries with LOCALIZED slugs the neutral path is not derivable from the URL — that is what the CMS `canonical_slug` link is for (see `localized-collections.md`): resolve the translated entry by its shared translation key, then link to that entry's URL. If a page has no counterpart in the other locale, point the switcher at that locale's home page rather than a 404 — and say so visually only if the brief wants it.

## Fallback semantics

`i18n.fallback` (e.g. `{ en: 'es' }`) makes untranslated routes exist in the secondary locale by reusing the default locale's content; on static output the fallback pages are generated at build time. `fallbackType: 'redirect'` (default) makes `/en/foo/` redirect to `/foo/`; `'rewrite'` serves the es content AT the /en/ URL.

Use fallback consciously as a launch bridge only:

- `redirect` is the honest default — the visitor sees the real (default-locale) URL.
- `rewrite` creates untranslated pages that LOOK translated; never emit hreflang for them as if they were translations, and prefer not shipping them at all on brand sites (a page in the wrong language under an /en/ URL reads as broken).

## Localized 404 and edge notes

- A single root `src/pages/404.astro` serves all locales on Workers Static Assets; render its strings via the dictionary keyed off the URL (see `ui-strings-dictionary.md`) since `Astro.currentLocale` on a 404 reflects the matched route, not the visitor's intent.
- If a brief ever forces `prefixDefaultLocale: true`, the root `/` on static output becomes a build-time redirect page — verify the emitted artifact in `dist/` before shipping, and prefer a `_redirects` rule (Workers Static Assets supports 2,000 static + 100 dynamic path rules) over a meta-refresh page.

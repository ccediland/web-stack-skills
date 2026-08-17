---
title: UI strings — the typed zero-dep dictionary and its escalation path
summary: The dictionary module pattern Astro's docs teach (ui.ts + getLangFromUrl + useTranslations), the Sveltia single_file data collection as the client-editable knob, and the Paraglide escalation with its threshold and caveats.
last_updated: 2026-08-17
applies_to: astro@7.2.2 · @inlang/paraglide-js@2.24.1 (escalation only) · Sveltia CMS i18n
---

# UI strings — the typed zero-dep dictionary

> Interface strings (nav, footer, buttons, aria-labels, form messages) are a small closed set on the sites this stack builds. One typed module covers them with zero dependencies, zero runtime JavaScript, and compile-time key checking. Everything heavier must be earned by string volume the brief can point at.

## Contents

- The dictionary module
- Using it in pages and components
- The CMS-editable knob (Sveltia single_file)
- Escalation — Paraglide, and only Paraglide
- Tombstones

## The dictionary module

```ts
// src/i18n/ui.ts
export const languages = { es: 'Español', en: 'English' } as const;
export const defaultLang = 'es';

export const ui = {
  es: {
    'nav.home': 'Inicio',
    'nav.contact': 'Contacto',
    'footer.rights': 'Todos los derechos reservados',
  },
  en: {
    'nav.home': 'Home',
    'nav.contact': 'Contact',
    'footer.rights': 'All rights reserved',
  },
} as const;
```

```ts
// src/i18n/utils.ts
import { defaultLang, ui } from './ui';

export function getLangFromUrl(url: URL) {
  const [, lang] = url.pathname.split('/');
  if (lang in ui) return lang as keyof typeof ui;
  return defaultLang;
}

export function useTranslations(lang: keyof typeof ui) {
  return function t(key: keyof (typeof ui)[typeof defaultLang]) {
    return ui[lang][key] ?? ui[defaultLang][key];
  };
}
```

This is the pattern Astro's own i18n recipe teaches. Properties that make it the default:

- `as const` gives every key compile-time existence checking — a typo in a key is a build error, not a blank string.
- The default-locale fallback in `t()` means a missing secondary-locale string degrades to the default language visibly, never to undefined.
- Zero bytes ship to the browser: the dictionary resolves during prerender.

## Using it in pages and components

```astro
---
import { getLangFromUrl, useTranslations } from '../i18n/utils';
const lang = getLangFromUrl(Astro.url);
const t = useTranslations(lang);
---
<a href="/">{t('nav.home')}</a>
```

`getLangFromUrl` (path-derived) works everywhere including 404 pages; `Astro.currentLocale` is equivalent on normal pages. Pick one per repo and stay consistent.

## The CMS-editable knob (Sveltia single_file)

When the brief wants the CLIENT to edit interface strings, keep the same shape but source it from a data file a Sveltia data collection edits:

- Store strings as JSON under the content tree (e.g. `src/content/strings/ui.json`) with the locales as top-level keys — exactly Sveltia's `single_file` i18n structure for data files.
- Declare a Sveltia collection over that file with `i18n: { structure: single_file }` and per-field `i18n: true`; the editor sees one entry with a locale switcher.
- The site imports the JSON into the same `useTranslations` helper; typing moves from `as const` to a zod schema if drift risk matters (same contract idea as data-layer's).

This knob composes with cms-self-edit (which owns the Sveltia setup itself); this skill only fixes the file shape both sides share.

## Escalation — Paraglide, and only Paraglide

Escalate when the dictionary stops being honest: past roughly 200 keys, when plurals and interpolation logic thicken, or when a translator workflow (external editor, machine-translation pipeline) enters the brief.

- Escalation target: `@inlang/paraglide-js` 2.x used DIRECTLY via its Vite plugin — compile-time, tree-shaken, typed messages.
- Caveat that must be re-verified at adoption time: Paraglide's canonical Astro setup assumes on-demand rendering with middleware; static/prerendered output is a documented but secondary mode where the locale is set during prerender instead of via `paraglideMiddleware()`. Prove it on a scratch page before committing a site to it.
- It adds an inlang project file and a compile step — real machinery. That is why it is the escalation, not the default, for sites whose string count fits in one screen of ui.ts.

## Tombstones

- `astro-i18next` — abandoned in beta since 2023 (still 1.0.0-beta.x). It remains the top tutorial hit for "Astro i18n library"; do not install it, do not follow tutorials built on it.
- `@inlang/paraglide-astro` — deprecated by inlang itself; paraglide-js 2.x is used directly, the Astro adapter package is explicitly "not needed anymore".
- `i18next` (runtime family) — alive upstream but a runtime-weight library with no maintained Astro glue; wrong weight class for prerendered marketing sites.

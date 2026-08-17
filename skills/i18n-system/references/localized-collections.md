---
title: Localized content collections — the folder layout Astro and Sveltia share
summary: The multiple_folders structure as the single convention for markdown collections, the glob-loader wiring and locale filtering, canonical_slug for linking translations, and partial-translation honesty rules.
last_updated: 2026-08-17
applies_to: astro@7.2.2 content layer (glob loader) · Sveltia CMS i18n structures
---

# Localized content collections

> One convention: markdown collections put each locale in its own subfolder (`src/content/blog/es/…`, `src/content/blog/en/…`). That layout is simultaneously Sveltia's first-class `multiple_folders` i18n structure and the layout Astro's own i18n recipe uses — the site and the CMS agree on disk with zero adapters. Data files (strings, settings) use `single_file` instead; see `ui-strings-dictionary.md`.

## Contents

- Layout and collection wiring
- Rendering localized routes
- Linking translations — canonical_slug
- Sveltia collection config
- Partial translations and placeholder discipline

## Layout and collection wiring

    src/content/blog/
      es/
        mi-primer-post.md
      en/
        my-first-post.md

ONE collection, locale encoded in the entry id (the subfolder becomes the id prefix):

```ts
// src/content.config.ts
const blog = defineCollection({
  loader: glob({ base: './src/content/blog', pattern: '**/*.md' }),
  schema: z.object({
    title: z.string(),
    date: z.date(),
    translationKey: z.string(), // shared across translations — see canonical_slug
  }),
});
```

Filter by locale at the call site:

```ts
const lang = Astro.currentLocale ?? 'es';
const posts = (await getCollection('blog')).filter((p) => p.id.startsWith(`${lang}/`));
```

One collection scales to new locales without touching config (a new subfolder just appears); per-locale collections with separate `base` paths also work but multiply config and schema declarations for nothing.

## Rendering localized routes

Default locale routes stay where they are (`src/pages/blog/[...slug].astro` filtering `es/`); the secondary locale gets its mirror under the locale folder (`src/pages/en/blog/[...slug].astro` filtering `en/`). Strip the locale prefix from the entry id when building the URL slug so `/blog/mi-primer-post/` and `/en/blog/my-first-post/` come out clean.

## Linking translations — canonical_slug

Translated entries have DIFFERENT slugs (localized titles), so nothing structural says which en post translates which es post. The link is an explicit frontmatter key shared by all translations of one piece:

- Sveltia's `i18n.canonical_slug` feature maintains exactly this — a `translationKey` field kept identical across locale variants of an entry, letting localized slugs diverge while the system knows they are one document.
- The language switcher for a collection page resolves the counterpart by `translationKey`, not by slug:

```ts
const counterpart = (await getCollection('blog')).find(
  (p) => p.id.startsWith(`${other}/`) && p.data.translationKey === entry.data.translationKey,
);
```

- The same key is what hreflang emission uses to pair alternate URLs for collection pages (see `hreflang-and-detection.md`). No counterpart found = no hreflang block and the switcher falls back to the locale's blog index.

## Sveltia collection config

The collection's `config.yml` entry declares the same structure so editors get a locale switcher instead of parallel collections:

```yaml
i18n:
  structure: multiple_folders
  locales: [es, en]
  default_locale: es
  canonical_slug:
    key: translationKey

collections:
  - name: blog
    folder: src/content/blog
    i18n: true
    fields:
      - { name: title, label: Title, i18n: true }
      - { name: date, label: Date, widget: datetime, i18n: duplicate }
```

Per-field `i18n` values: `true` (translated per locale), `duplicate` (same value copied to all locales — dates, images), `false` (default locale only). cms-self-edit owns the rest of the Sveltia setup; verify field-level details against the Sveltia i18n docs at wiring time — it is a beta-cadence project.

## Partial translations and placeholder discipline

- A collection is ALLOWED to be partially translated; the routing, switcher, and hreflang recipes above all degrade correctly when a counterpart is missing. What is not allowed is faking coverage: no empty en files, no machine-translated brand voice committed as if ratified.
- Brand-voice rule (from the interchange contract seam): the brand canon defines voice per locale; a locale the canon does not cover has NO brand voice to write in. Fixtures and staging mark secondary-locale copy as visible placeholders; real copy is an owner deliverable, not a build artifact.
- `i18n.fallback` interacts here: `rewrite` would make partially-translated collections look fully translated. Keep collections on the explicit-file model and leave fallback to page routes, if used at all.

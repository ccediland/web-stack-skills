---
title: Sveltia setup — mount, config.yml, and Astro content collections
summary: How to mount the Sveltia admin under public/admin, write config.yml for the GitHub backend, and map Sveltia collections to Astro 6 content collections and their Zod schemas.
last_updated: 2026-06-18
applies_to: Sveltia CMS 0.167.2 on Astro 6.x with @astrojs/cloudflare 13.x, GitHub backend
---

# Sveltia setup — mount, config.yml, and Astro content collections

> Mount the Sveltia admin as a static file under public/admin, declare the GitHub backend in config.yml, and map each Sveltia collection onto an Astro content collection so saves produce frontmatter Astro parses.

## Contents
- Mounting the admin
- The config.yml skeleton
- Mapping collections to Astro content collections
- Field types to Zod
- Slugs, dates, and the body field
- Gotchas

## Mounting the admin

Install the bundle as a first-party dependency so the version is pinned and the CSP needs no third-party origin. The CDN script from unpkg works too, but then unpkg has to be allowed in script-src and connect-src, and the version floats.

```bash
npm install @sveltia/cms
```

Create public/admin/index.html. The robots noindex keeps the admin out of search. Loading the npm bundle as a module keeps it first-party.

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="robots" content="noindex" />
    <title>Content</title>
  </head>
  <body>
    <script type="module" src="/admin/sveltia-cms.js"></script>
  </body>
</html>
```

Copy node_modules/@sveltia/cms/dist/sveltia-cms.js into public/admin/ during the build (a prebuild script or a copy step), so it ships as a first-party asset rather than a bare import. Anything under public/ is a passthrough asset on Astro: it is not processed, so Astro's CSP meta element is never injected into it.

## The config.yml skeleton

Put config.yml next to index.html, at public/admin/config.yml. The branch is drafts so saves never hit production directly (see i18n-and-publishing.md). base_url points at the OAuth Worker (see cloudflare-auth-worker.md); omit it only if every editor uses a personal access token.

```yaml
backend:
  name: github
  repo: ccediland/<repo>
  branch: drafts
  base_url: https://sveltia-cms-auth.<subdomain>.workers.dev
media_folder: public/media
public_folder: /media
output:
  quote: double
collections:
  - name: blog
    label: Blog
    label_singular: Post
    folder: src/content/blog
    create: true
    format: yaml-frontmatter
    slug: '{{year}}-{{month}}-{{day}}-{{slug}}'
    fields:
      - { label: Title, name: title, widget: string }
      - { label: Date, name: date, widget: datetime }
      - { label: Description, name: description, widget: text }
      - { label: Draft, name: draft, widget: boolean, default: true, required: false }
      - { label: Tags, name: tags, widget: list, required: false }
      - { label: Body, name: body, widget: markdown }
```

The output.quote option replaces the deprecated per-collection yaml_quote and controls how string values are quoted in YAML, which matters for frameworks that are strict about frontmatter.

## Mapping collections to Astro content collections

Astro 6 reads content through the content layer: a glob loader points at a directory, and a Zod schema validates each entry's frontmatter. One Sveltia folder collection maps to one Astro collection directory.

```ts
// src/content.config.ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const blog = defineCollection({
  loader: glob({ pattern: '**/[^_]*.{md,mdx}', base: './src/content/blog' }),
  schema: z.object({
    title: z.string(),
    date: z.coerce.date(),
    description: z.string(),
    draft: z.boolean().default(false),
    tags: z.array(z.string()).optional(),
  }),
});

export const collections = { blog };
```

The Sveltia folder path (src/content/blog) is the same directory the glob loader reads. Every field declared in Sveltia must have a matching schema entry, or Astro's build fails validation.

## Field types to Zod

| Sveltia widget | Zod schema |
|---|---|
| string, text | z.string() |
| datetime | z.coerce.date() |
| boolean | z.boolean() |
| number | z.number() |
| select (single) | z.enum([...]) |
| list | z.array(...) |
| object | z.object({...}) |
| relation | reference('other-collection') |
| markdown (body field) | rendered body, not a frontmatter field |

Use z.coerce.date() for datetime because Sveltia writes an ISO 8601 string and Astro coerces it to a Date. Make a field optional in Zod only if it is required: false in Sveltia, or the build breaks on entries that omit it.

## Slugs, dates, and the body field

A field named body is treated as the entry content and written below the frontmatter, not as a frontmatter key. The Astro entry id is the filename slugified; override it with the slug template. Available templates include {{slug}}, date parts like {{year}}-{{month}}-{{day}}, and {{uuid}} or {{uuid_short}} or {{uuid_shorter}} for non-Latin titles. Set identifier_field to change which field the slug is derived from.

## Gotchas

- Do not set format to toml. Sveltia's TOML frontmatter output is unreliable; YAML frontmatter is the safe default and is what Astro expects.
- A Sveltia field with no matching Zod key, or a Zod key with no default that Sveltia leaves empty, fails the Astro build. Keep the two in sync.
- For a relation field on an i18n site, the saved value may lack the locale prefix. Set value_field to include the locale, for example '{{locale}}/{{slug}}', so the reference resolves. See [Sveltia discussion 302](https://github.com/sveltia/sveltia-cms/discussions/302): the Astro i18n relation fix.
- yaml_quote per collection is deprecated and removed at 1.0.0. Use the top-level output.quote.

## References
- [Entry Collections | Sveltia CMS](https://sveltiacms.app/en/docs/collections/entries): collection, folder, and field reference.
- [Content collections | Astro Docs](https://docs.astro.build/en/guides/content-collections/): the glob loader and Zod schema model.
- [Fields | Sveltia CMS](https://sveltiacms.app/en/docs/fields): widget types and per-field options.

---
title: Build-time loaders — file() tier zero and the custom Supabase loader
summary: The two build-time recipes — the file() loader over a seed JSON and a custom content-collection loader reading Supabase at build time — with the single zod schema that types both, digest-based incremental updates, and the parseData validation seam.
last_updated: 2026-08-17
applies_to: astro@7.2.2 (Content Loader API, astro/loaders) · @supabase/supabase-js@2.112.3
---

# Build-time loaders

> Both tiers hang off the same two primitives: Astro's Content Loader API (`astro/loaders`) and one zod schema. Tier zero reads a committed seed file; tier one reads Supabase at build time. Swapping tiers changes the loader line, never the schema, never the pages.

## Contents

- The shared schema
- Tier zero — file() over the seed
- Tier one — custom Supabase loader
- Incremental updates (digest + meta)
- Validation semantics
- Gotchas

## The shared schema

One zod schema, hand-written, in its own module so every consumer imports the same object. Astro 7 bundles zod 4 and exposes it as `astro/zod` — no separate dependency.

```ts
// src/data/catalog-schema.ts
import { z } from 'astro/zod';

export const catalogItem = z.object({
  id: z.string(),
  name: z.string(),
  price_mxn: z.number().int().nonnegative(),
  category: z.string(),
  available: z.boolean().default(true),
  sort: z.number().int().default(0),
});
export type CatalogItem = z.infer<typeof catalogItem>;
```

This module is the data CONTRACT: the seed file, the Supabase table, and any future consumer conform to it. When the site consumes a governed brand repo, this is the schema its data-map names (see `brand-contract-seam.md`).

## Tier zero — file() over the seed

```ts
// src/content.config.ts
import { defineCollection } from 'astro:content';
import { file } from 'astro/loaders';
import { catalogItem } from './data/catalog-schema';

const catalog = defineCollection({
  loader: file('data/catalog.json'),
  schema: catalogItem,
});
export const collections = { catalog };
```

`file()` expects an array of objects with unique `id` fields (or an object keyed by id). Pages consume via `getCollection('catalog')` — and that call site never changes again, whatever tier the source moves to.

## Tier one — custom Supabase loader

An inline loader is an async function; the object form adds incremental updates. Runs at BUILD time only — nothing here ships to the browser.

```ts
// src/data/supabase-loader.ts
import type { Loader } from 'astro/loaders';
import { createClient } from '@supabase/supabase-js';
import { catalogItem } from './catalog-schema';

export function catalogLoader({ url, key }: { url: string; key: string }): Loader {
  return {
    name: 'catalog-supabase',
    load: async ({ store, parseData, generateDigest, logger }) => {
      const supabase = createClient(url, key);
      const { data, error } = await supabase
        .from('catalog_items')
        .select('*')
        .order('sort', { ascending: true });
      if (error) throw new Error(`catalog load failed: ${error.message}`);

      store.clear();
      for (const row of data) {
        const parsed = await parseData({ id: String(row.id), data: row });
        store.set({ id: String(row.id), data: parsed, digest: generateDigest(parsed) });
      }
      logger.info(`catalog: ${data.length} items`);
    },
    schema: catalogItem,
  };
}
```

```ts
// src/content.config.ts (tier-one wiring)
const catalog = defineCollection({
  loader: catalogLoader({
    url: import.meta.env.SUPABASE_URL,
    key: import.meta.env.SUPABASE_PUBLISHABLE_KEY,
  }),
});
```

The publishable key is safe in build env vars (it is the browser-grade key; RLS governs it) — but at tier one the browser never sees it, because the query runs in the build. A read-only RLS policy on the table is still the right posture: the key's blast radius stays read-only even if it leaks.

## Incremental updates (digest + meta)

`store.set()` returns false and skips the write when the entry's `digest` matches what is stored — unchanged rows cost nothing downstream. `generateDigest()` produces the non-cryptographic digest for exactly this. The loader `meta` KV persists between builds (sync tokens, a last-modified watermark) — useful when the dataset grows enough that full reloads hurt; for a catalog of dozens to hundreds of rows, `store.clear()` + full reload is simpler and honest.

## Validation semantics

`parseData` validates every entry against the collection schema AT BUILD TIME: a row that violates the contract FAILS THE BUILD with the offending id in the error. This is the property that makes one zod schema worth more than generated types — generated TypeScript types vanish at runtime and validate nothing. Keep Supabase generated types (`supabase gen types`) as a drift cross-check in code review if you like; the zod schema remains the enforcement point.

## Gotchas

- Loader errors abort the build — that is correct behavior (a site silently built without its catalog is worse than a failed deploy), but it means the rebuild webhook can produce a failed build on bad data; the Deploy Hook dedup keeps retries sane (see `rebuild-wiring.md`).
- The supabase-js client is a devDependency-grade cost at tier one (build-time only); it must NOT appear in any client bundle. If the CI byte budget suddenly shows it, something imported it from an island — that is a tier jump by accident, catch it in review.
- `file()` parses JSON natively; for CSV or other formats pass a `parser` function.
- IDs: content-collection ids are strings — cast numeric primary keys with `String(row.id)` consistently or `getEntry` lookups drift.

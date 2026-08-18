---
title: Content modeling — from brief to collection schema and its config.yml mirror
summary: The discipline of deciding what becomes a content collection, designing the Zod schema first, mirroring it in Sveltia's config.yml, and the documents-versus-rows boundary with data-layer — added as a W4 extension.
last_updated: 2026-08-18
applies_to: astro@7.2.2 content layer (glob loader, astro/zod) · @sveltia/cms config.yml · sibling skill data-layer (rows)
---

# Content modeling

> Modeling happens BEFORE fields get typed into config.yml: decide what is a collection at all, whether it is documents or rows, and write the Zod schema as the single contract — the CMS config is its mirror, never a second source of truth. Most modeling mistakes are not wrong field types; they are wrong HOMES (a catalog modeled as markdown, a page modeled as a collection) and unmirrored drift between schema and config.

## Contents

- Documents versus rows — the boundary decision
- What becomes a collection
- Schema-first, config-mirror
- Field modeling rules
- Singletons and page content
- Localized collections
- Drift discipline

## Documents versus rows — the boundary decision

The first modeling question is not "what fields" but "which sibling owns this":

| Content shape | Home | Why |
|---|---|---|
| Prose a human edits as a page or post (blog, guías, casos) | THIS skill — markdown/YAML files, Sveltia edits them | Documents want history, review, and an editor UI over files |
| Business data with identity and lifecycle (catalog, plans, prices, inventory) | data-layer — seed JSON or Supabase rows behind ONE Zod contract | Rows want validation, relational integrity, and non-web consumers |
| Prose ABOUT a row (a long description attached to a plan) | The row keeps a key; the prose lives as a document carrying that key | Same split i18n uses (translationKey) — join by key at build |

The tell for a mis-homed model: if editors would ever need "add a field to every entry at once" or another system reads the same data, it was rows. If the client asks to "write and preview" it, it was documents. The fixture's `planes` catalog is the standing example of the rows side; its blog is the documents side.

## What becomes a collection

- A COLLECTION is content with many entries of the same shape and a listing surface (posts, guides, testimonials, FAQ entries).
- Not everything the client edits is a collection: one-off page copy is a SINGLETON (below); site-wide settings (contact info, hours) are one settings file, not N.
- The unit of entry is the unit of publishing — if two things always publish together, they are one entry with two fields, not two entries.
- Resist speculative fields: model what current content needs; the schema grows by edit, and additive schema changes are cheap (add optional, backfill later). Removing or renaming is migration work — see drift discipline.

## Schema-first, config-mirror

Order of authorship, always:

1. Write the Zod schema in `src/content.config.ts` (via `astro/zod`) — types, required-versus-optional, defaults, constraints. This is the CONTRACT; the build enforces it on every entry.
2. Mirror it in `public/admin/config.yml` — one Sveltia collection per content collection, fields matching the schema one for one (the field-to-Zod table lives in sveltia-setup.md).
3. Round-trip test: create an entry through the admin, run the build; the build failing on a CMS-authored entry means the mirror drifted — fix the config, never loosen the schema to make bad content pass.

The schema is stricter than the CMS on purpose: config.yml `required: true` is a UI hint; `z.string().min(1)` is a wall. Editors meet the wall at build time via the drafts-branch CI check (the publishing flow this skill already ships).

## Field modeling rules

- Every entry gets a stable identity the URL derives from — the file slug. Fields that look like identity (title) are display, not identity; renaming a title must not move a URL.
- Dates are explicit fields (`z.coerce.date()`), never inferred from filenames only — filename dates are a sorting convenience, the field is the truth.
- Enums over free strings for anything that drives rendering (`z.enum(['guide','case'])` versus a `category` free-text that fragments into "Guía"/"guia"/"guías").
- Booleans that gate visibility (`draft`, `activo`) default to the SAFE side (`draft: true` — new content hides until flipped).
- Images referenced by collection entries go through the media decision this skill already owns (repo media versus R2); the schema types them as paths/URLs, and alt text is a REQUIRED sibling field (a11y-deep's floor applies to CMS content too).
- Relations across collections are keys, not embeds (`relatedGuides: z.array(z.string())` of slugs) — resolve at build; embedding copies is how content forks.

## Singletons and page content

Sveltia's file collections model one-off editable surfaces: the homepage hero copy, the about page, a settings file. Each file gets its own schema block. The rule for WHICH page copy becomes editable: only copy the client will actually re-touch (offers, hours, announcements). Structural copy that changes only with redesigns stays in the components — every editable surface is a support surface, and the marked-placeholder discipline applies to empty ones.

## Localized collections

Localized content models ONCE and mirrors per locale: the i18n-system pattern (multiple_folders + translationKey as Sveltia's canonical_slug) is the standing layout — the schema adds `translationKey: z.string()` and nothing else changes shape. Model in the default locale first; the second locale is a projection of the same schema, never a divergent one.

## Drift discipline

- Additive change (new optional field): schema first, config second, backfill at leisure. Safe.
- Breaking change (rename, retype, required-ify): migrate the content files in the SAME commit as the schema change — the build is the enforcement, so a schema-only commit breaks every existing entry at once. Grep-and-edit across `src/content/` is the migration tool at this scale; a script earns its place past ~50 entries.
- config.yml and the schema live in different files owned by different mindsets (editor UX versus build contract) — when they disagree, the schema wins and the config is the bug. Re-run the round-trip test after every schema change; it is the cheapest drift detector this layer has.

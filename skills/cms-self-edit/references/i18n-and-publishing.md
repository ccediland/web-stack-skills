---
title: i18n, AI translation, and the drafts-branch publishing flow
summary: Configure Spanish and English locales with the multiple_folders structure, wire a currently-working AI translation service (not DeepL), and publish through a drafts branch with CI and a Cloudflare preview URL.
last_updated: 2026-06-18
applies_to: Sveltia CMS i18n and single-branch publishing, Astro 6 on Cloudflare Workers Static Assets
---

# i18n, AI translation, and the drafts-branch publishing flow

> For a Spanish and English site, use multiple_folders with default_locale es so locales map to Astro's per-locale routing. Translate with Google Cloud Translation, Gemini, or Mistral, because DeepL is currently disabled. Publish by committing to a drafts branch, gating with CI, and merging to production, with a Cloudflare preview URL for the client.

## Contents
- i18n structure for Spanish and English
- Field-level i18n
- AI translation, and the DeepL status
- Publishing on a single branch
- The drafts-branch flow for a non-technical client
- Preview on Workers Static Assets
- Gotchas

## i18n structure for Spanish and English

i18n is declared at the top level, the collection level, and the field level. For an Astro Spanish and English site, use multiple_folders, which stores each locale in its own subfolder per collection and maps cleanly to Astro's src/pages per-locale routing. Set default_locale to es if Spanish is canonical.

```yaml
i18n:
  structure: multiple_folders
  locales: [es, en]
  default_locale: es
collections:
  - name: blog
    folder: src/content/blog
    i18n: true
    fields:
      - { label: Title, name: title, widget: string, i18n: true }
      - { label: Date, name: date, widget: datetime, i18n: duplicate }
      - { label: Body, name: body, widget: markdown, i18n: true }
```

The structures are single_file, multiple_files (a locale suffix on the filename), multiple_folders (a subfolder per locale), and multiple_root_folders (a separate root tree per locale, which replaces the deprecated multiple_folders_i18n_root). multiple_files is simpler but reads less cleanly against Astro's folder routing; multiple_root_folders fits when each language is an independent tree. omit_default_locale_from_filename drops the locale code from the default-locale filenames.

## Field-level i18n

The per-field i18n value controls how a field behaves across locales:

| Value | Behavior |
|---|---|
| true | edited and stored separately per locale |
| duplicate | edited in the default locale and copied to the others |
| false | edited only in the default locale |

For Decap compatibility, translate and none are accepted as aliases of true and false. Sveltia links localized entries automatically by filename or folder, so the editor sees one entry with a locale switch, not parallel files.

## AI translation, and the DeepL status

DeepL is currently disabled in Sveltia, removed over an API limitation and possibly returning with a CORS-proxy workaround. Do not configure or promise DeepL. The currently supported services are an NMT, Google Cloud Translation, and several LLMs (Anthropic, Gemini, Mistral, OpenAI, DeepSeek).

Recommended default: Google Cloud Translation, whose first 500,000 characters per month are free. The editor opens an entry, clicks Translate on the pane header (which fills only empty fields) or on a field (which overrides), picks the service, and pastes the API key when prompted. The key is stored in the browser local storage. For multiple editors, the admin distributes one key, since asking a non-technical client to create a cloud API key is a poor experience.

If the translation service is called from the admin, add its endpoint to the /admin CSP connect-src (for Google Cloud Translation, https://translation.googleapis.com).

## Publishing on a single branch

Sveltia commits to one branch; Editorial Workflow, which would give a branch per entry, does not exist yet. Point the backend at a drafts branch in config.yml (backend.branch: drafts). Saves commit there, never to production directly. Because there is one branch, only one set of drafts is in flight at a time, and the next merge publishes everything on drafts.

## The drafts-branch flow for a non-technical client

1. The client edits in Sveltia and saves. Sveltia commits to drafts.
2. The project CI (ci.yml, owned by perf-ci-gates) runs on the drafts branch.
3. To publish, drafts is merged into the production branch. Have the developer do the merge, or open a pull request with auto-merge once CI is green. The client does not need to touch git if the developer merges.

Commit-to-rebuild latency depends on the Astro build plus the Cloudflare build queue; measure it on the real site before promising the client a number. There is no reliable published figure.

## Preview on Workers Static Assets

Cloudflare Pages has per-branch preview deployments; Workers Static Assets does not replicate that automatically, but it offers preview URLs. Enable Workers Builds, connect the repo, and turn on non-production branch builds (Settings, Build, Branch control). A commit to drafts then runs a versions upload and produces a preview URL; an aliased preview URL such as drafts-WORKER.SUBDOMAIN.workers.dev is persistent per branch. The client opens that URL to see the drafts branch rendered before the merge. Protect it with Cloudflare Access if it should not be public. Preview URLs are not generated for Workers using Durable Objects.

## Gotchas

- Do not reference DeepL anywhere; it is disabled. Use Google Cloud Translation or another listed service.
- Translation API keys live in browser local storage, so each new browser re-enters the key.
- With one branch, two concurrent drafts publish together at the next merge. If editors need isolated drafts, that is the boundary toward Editorial Workflow (not yet shipped) or a heavier CMS.
- Preview URLs have no request logs and are skipped for Durable Object Workers.

## References
- [Internationalization (i18n) | Sveltia CMS](https://sveltiacms.app/en/docs/i18n): structures, default locale, and field-level i18n.
- [Translation Services | Sveltia CMS](https://sveltiacms.app/en/docs/integrations/translations): the current service list and the DeepL status.
- [Editorial Workflow | Sveltia CMS](https://sveltiacms.app/en/docs/workflows/editorial): the not-yet-implemented status that forces the drafts-branch stand-in.
- [Preview URLs | Cloudflare Workers](https://developers.cloudflare.com/workers/configuration/previews/): per-version and aliased preview URLs.
- [Builds | Cloudflare Workers](https://developers.cloudflare.com/workers/ci-cd/builds/): non-production branch builds.

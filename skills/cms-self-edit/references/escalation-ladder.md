---
title: Escalation ladder — Pages CMS and Directus
summary: When Sveltia is outgrown, the decision criteria for moving to Pages CMS (magic-link, client never sees GitHub) and then to Directus (relational or multi-channel content), with current licensing.
last_updated: 2026-06-18
applies_to: git-based CMS to database CMS escalation for Astro 6 on Cloudflare
---

# Escalation ladder — Pages CMS and Directus

> Stage 1 is Sveltia. Move to Stage 2, Pages CMS, when the client must never see GitHub and needs magic-link login. Move to Stage 3, Directus, when content turns relational or multi-channel, needs instant publish, or needs a real editorial workflow. This file gives the criteria only, not full recipes.

## Contents
- Stage 1 to Stage 2 — Pages CMS
- Stage 2 to Stage 3 — Directus
- The git-versus-database line
- Criteria summary

## Stage 1 to Stage 2 — Pages CMS

Move to [Pages CMS](https://github.com/pages-cms/pages-cms) when the client must not see GitHub at all and needs passwordless login. Pages CMS is still git-based (content stays as files in the repo), but it invites contributors by email and sends a passwordless magic-link login, so an editor needs no GitHub account.

The cost is a reintroduced dependency. The hosted version at app.pagescms.org is a third-party SaaS that you authorize against your repo through its GitHub App, which means a third party has read and write access to the content. The alternative is self-hosting, which adds Postgres and BetterAuth (DATABASE_URL, BETTER_AUTH_SECRET, CRYPTO_KEY) plus your own GitHub App. Pages CMS is MIT-licensed and free either way. For privacy and security, prefer self-host in production.

Take Stage 2 only when magic-link or no-GitHub onboarding is the deciding need, and you accept either the third-party SaaS with repo access or the cost of running Postgres and BetterAuth.

## Stage 2 to Stage 3 — Directus

Move to [Directus](https://directus.com) when the content stops being flat files for one static site. Directus is database-backed (Postgres) and is a per-site engine, not a drop-in. It is warranted when content is relational (many links between content types), multi-channel (several frontends consuming one API), needs instant publish without a static rebuild, or needs a real editorial workflow with roles and approvals.

Licensing, current as of the v12 change: Directus moved to the [Monospace Sustainable Core License](https://directus.com/resources/directus-v12-license-change) (MSCL 1.0, a Fair Core derivative, replacing the earlier BUSL). Free commercial use is available through the Open Innovation Grant for an entity with less than 5 million dollars in annual revenue and fewer than 50 employees. Above that threshold there is a limited free Core tier or a commercial license for enterprise features. Each version becomes GPLv3 four years after release, and the SDKs remain MIT.

## The git-versus-database line

Git-based (Sveltia, Pages CMS) is right when content is flat files versioned for a static site: pages, posts, marketing copy, with simple relations. The repo is the database, there is no server to run, and content has no lock-in.

Database-backed (Directus) is right when the content model itself needs a database: deep relations, querying across types, an API consumed by more than one client, publishing that cannot wait for a static rebuild, or a workflow with roles. At that point the file-per-entry model fights the content rather than serving it.

## Criteria summary

| Move | Trigger | Cost accepted |
|---|---|---|
| Stage 1 to 2 | Client must not see GitHub; needs magic-link login | Third-party SaaS with repo access, or self-host Postgres and BetterAuth |
| Stage 2 to 3 | Content is relational or multi-channel; instant publish; roles and approvals | A per-site Postgres engine, hosting cost, MSCL licensing terms |

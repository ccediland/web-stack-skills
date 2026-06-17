---
name: stack-integration-playbook
description: Deferred skeleton — documents how all parts of a website stack compose and operate together (tokens, motion, WebGL, schema, CSP, CI, island hydration, load order, seams) and how the site connects to the wider stack beyond the web (database, repos, CDN, CI runners, secrets, workspace, comms, payments, catalogs, ads, social, analytics). Skeleton only for now; filled over time with field-learned integration lessons. Not part of the installable plugin until it has substance.
---

# stack-integration-playbook

> Status — DEFERRED SKELETON. This skill lives in `deferred/` (outside `skills/`) and is intentionally excluded from the installable plugin until it has substance. It does not pass the 5-turn cadence yet.

## Purpose

Document how every element of the site executes and interacts — how tokens, motion, WebGL, schema, CSP, and CI compose; how islands hydrate; load order and seams — and the functions and interactions with the rest of the stack beyond the web (Supabase, other repos, Cloudflare, GitHub Actions, Infisical, Google Workspace, comms, payments, catalogs, ads, social, analytics).

## Outline (filled much later, from field lessons)

- How the 7 skills compose on one real site (load order, hydration, seams).
- Cross-cutting gotchas discovered while building and operating the stack.
- Web to rest-of-stack integration points (data, auth, secrets, deploy, comms, payments, analytics).

## Notes

- Living doc — populated with lessons learned the hard way; the natural home for cross-cutting gotchas.
- Generic only — project-specific IDs, accounts, and endpoints belong in the consumer project's RESIDENT, not here.

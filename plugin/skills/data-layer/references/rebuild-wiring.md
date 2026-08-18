---
title: Rebuild wiring — Supabase webhook to Cloudflare Deploy Hook
summary: The native freshness cycle for build-time data — a Supabase Database Webhook (pg_net) POSTing a Workers Builds Deploy Hook, with the hook URL secured in Supabase Vault, plus the repository_dispatch variant for gated rebuilds.
last_updated: 2026-08-17
applies_to: Cloudflare Workers Builds Deploy Hooks (GA 2026-04) · Supabase Database Webhooks (pg_net) · Supabase Vault
---

# Rebuild wiring

> Tier-one freshness is a rebuild, and the native path is two managed primitives talking directly: a database trigger fires an async HTTP POST at a per-branch Deploy Hook URL, and Workers Builds rebuilds from the same commit — the loader re-fetches fresh rows. No GitHub Actions hop, no data snapshot, no polling.

## Contents

- The Deploy Hook
- The Supabase webhook
- Securing the hook URL (Vault)
- Burst behavior and failure modes
- The repository_dispatch variant
- Verification

## The Deploy Hook

Create it in the Cloudflare dashboard (Workers & Pages → the Worker → Settings → Builds → Deploy Hooks) or via API, bound to ONE branch. The result is a unique URL:

```
POST https://api.cloudflare.com/client/v4/workers/builds/deploy_hooks/DEPLOY_HOOK_ID
```

An unauthenticated POST triggers build + deploy of the git-connected Worker for that branch. Properties that matter here (platform-verified 2026-08-17): no Authorization header needed — THE URL IS THE CREDENTIAL; automatic deduplication while a build is queued or initializing; rate limits of 10 builds/min per Worker and 100/min per account. Requires a git-connected Worker (Workers Builds), which this stack already runs.

## The Supabase webhook

Database Webhooks are a managed wrapper over Postgres triggers using the pg_net extension — async and non-blocking, so the INSERT/UPDATE/DELETE that fired it never waits on the HTTP call. Create in Dashboard (Database → Webhooks) or SQL. Fire it on the table(s) the loader reads, on all three events.

Do NOT paste the hook URL into the trigger definition — read it from Vault at call time (next section):

```sql
create or replace function public.trigger_site_rebuild()
returns trigger language plpgsql security definer as $$
declare hook_url text;
begin
  select decrypted_secret into hook_url
    from vault.decrypted_secrets where name = 'deploy_hook_url';
  perform net.http_post(url := hook_url, body := '{}'::jsonb);
  return coalesce(new, old);
end $$;

create trigger catalog_rebuild
  after insert or update or delete on public.catalog_items
  for each statement execute function public.trigger_site_rebuild();
```

`for each statement` (not `for each row`) keeps a bulk edit to one webhook call per statement; Deploy Hook dedup absorbs whatever still bursts.

## Securing the hook URL (Vault)

An unauthenticated URL that deploys your site is a secret with a deploy-grade blast radius. Store it in Supabase Vault (`vault.create_secret('https://api.cloudflare.com/.../DEPLOY_HOOK_ID', 'deploy_hook_url')`) and read it inside the trigger function, per Supabase's own documented pattern. Rotation = delete the hook in Cloudflare, create a new one, update the Vault secret — no SQL redeploy. The URL also belongs in the project's secrets manager inventory (name, not value, in the repo docs).

## Burst behavior and failure modes

- Bulk data edits: statement-level trigger + hook dedup + 10/min rate limit means the worst case is one extra queued build, not a build storm.
- Bad data: the loader fails the build on a row that violates the schema (by design). The failed build does NOT take the live site down — the previous deploy keeps serving; fix the row, the next webhook (or a manual POST) rebuilds. Watch build notifications for exactly this case.
- Webhook loss: pg_net is fire-and-forget; a lost POST means stale-until-next-change. If that matters, add a low-frequency scheduled POST to the same hook as a freshness floor (a cron anywhere that can POST — even a GitHub Actions schedule — but keep it as floor, not primary).

## The repository_dispatch variant

Choose GitHub Actions as the intermediary ONLY when a rebuild must earn something the direct hook cannot give:

- CI gates before deploy: webhook → `repository_dispatch` → workflow runs the gates → on green, POSTs the Deploy Hook (or pushes a commit). Costs a PAT (fine-grained, single-repo) as a Supabase Vault secret and an extra hop of latency.
- Committed data snapshot for audit: the workflow fetches the dataset, commits it (which itself triggers Workers Builds), giving a git history of every data state that ever shipped.

Default remains the direct hook — native-first, fewer moving parts, no PAT.

## Verification

Wire it, then prove the loop ONCE end-to-end before trusting it: edit a row → confirm the webhook fired (Supabase webhook logs) → confirm a build started and completed (Workers Builds history) → confirm the deployed page shows the new value. A wiring that has never moved a real row to a real page is a diagram, not a pipeline.

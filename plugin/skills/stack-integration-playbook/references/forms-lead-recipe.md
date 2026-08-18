---
title: Forms and lead capture — the one-route native recipe
summary: The catalog's lead-capture architecture — an Astro Action on the site's own Worker, Turnstile anti-spam, Supabase insert-only storage, native email notification, LFPDPPP consent, and the WhatsApp deep-link knob — with per-brief knobs and the verbatim flip conditions that would promote this recipe to a full skill.
last_updated: 2026-08-18
applies_to: astro@7.2.2 (Actions, astro/zod) · @astrojs/cloudflare@14.2.1 · Cloudflare Turnstile + Email Service (public beta) · @supabase/supabase-js@2.112.3
---

# Forms and lead capture — the one-route native recipe

> This is a RECIPE, not a skill, by the catalog's own criterion: the technically-best path converges on exactly ONE native route — an Astro Action on the Worker the site already deploys — and everything that varies per brief is a knob on that architecture, not a competing architecture. The flip conditions at the end are the standing test; if one fires, this recipe is re-scoped as the `forms-lead-system` skill.

## Contents

- The architecture in one view
- Why the alternatives lost
- The deploy-shape delta
- The Action
- Turnstile
- Storage — Supabase insert-only
- Email notification — the native slot
- LFPDPPP consent (Mexico)
- The WhatsApp knob
- Abuse controls beyond Turnstile
- Gate amendments (CSP + budgets)
- Knobs per brief
- Flip conditions (verbatim, standing)

## The architecture in one view

Browser form → client RPC to the site's own `/_actions/*` endpoint (Astro Action; pages stay prerendered) → server-side Turnstile siteverify → zod-validated insert into a Supabase `leads` table (insert-only RLS, publishable key) → native email notification to the owner's verified inbox → optional downstream CRM sync later. Anti-spam: Turnstile + honeypot; consent: LFPDPPP checkbox stored with the lead.

## Why the alternatives lost

Evaluated on merit (primary-source research, 2026-08-17): a SEPARATE form Worker is strictly dominated — its only rationale, keeping the site's Worker assets-only, is not a real constraint because the adapter serves mixed prerendered+on-demand first-class. THIRD-PARTY form backends (Web3Forms/Formspree/Basin) lose on thin free tiers (50–250 subs/mo), a costlier CSP (their origin + their captcha origins), Mexican lead data in a foreign SaaS (LFPDPPP transfer friction), and no path into Supabase/CRM without extra glue. A WHATSAPP-ONLY deep link is not an architecture but a UX option that coexists (see the knob).

## The deploy-shape delta

Adding the Action is the moment the site's Worker gains a `main` entry next to its assets binding — pages remain prerendered; only `/_actions/*` renders on demand (proven on a live Workers Builds branch deploy, 2026-08-17). Consequences to wire consciously: the CSP on server-rendered responses arrives as a HEADER (not the meta element) — the Action returns JSON, so in practice nothing changes for pages; CI's audited set only grows by the contact page itself.

Two field-proven gotchas of this delta:

- The split-config pattern. Wrangler must see `main` (plus `assets` and `build.command`) in the config it parses so `versions upload` on Workers Builds knows what to deploy — wrangler runs the custom build FIRST, then resolves the paths it creates. But the Cloudflare Vite plugin inside `astro build` errors on a `main` that does not exist yet, and astro clears the output dir before the plugin resolves config, so a stub file cannot survive. Resolution: the adapter takes `configPath` — point the PLUGIN at a minimal `wrangler.vite.jsonc` (name, compatibility, NO main) and keep the root `wrangler.jsonc` as the deploy config with main + assets + build.command.
- Worker env access: `Astro.locals.runtime.env` was REMOVED in Astro v6 — read secrets via `import { env } from 'cloudflare:workers'` (the adapter externalizes the module). The old pattern fails only at runtime on workerd, so test the action on `astro preview`, not just the build.

## The Action

```ts
// src/actions/index.ts
import { defineAction } from 'astro:actions';
import { z } from 'astro/zod';

export const server = {
  lead: defineAction({
    accept: 'form',
    input: z.object({
      name: z.string().min(2).max(120),
      contact: z.string().min(5).max(160),        // email or phone — one field, MX habit
      message: z.string().max(2000).default(''),
      consent: z.coerce.boolean().refine((v) => v === true, 'consent required'),
      website: z.string().max(0).optional(),       // honeypot — any value rejects
      turnstile: z.string().min(1),                // cf-turnstile-response token
    }),
    handler: async (input, ctx) => {
      // 1 siteverify → 2 insert → 3 notify (below)
    },
  }),
};
```

`accept: 'form'` parses FormData by input name and coerces the checkbox; validation failure returns BAD_REQUEST before the handler runs. Call it client-side (`actions.lead(new FormData(form))`) from a small bundled script — the zero-JS `<form action>` fallback would force the PAGE on-demand, which this recipe deliberately avoids.

## Turnstile

Managed widget (default), explicit render from a bundled script; `appearance: 'interaction-only'` keeps it invisible until a challenge is needed. Server-side verification is MANDATORY (an unverified widget is decoration):

```ts
const verify = await fetch('https://challenges.cloudflare.com/turnstile/v0/siteverify', {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({
    secret: ctx.locals.runtime.env.TURNSTILE_SECRET,
    response: input.turnstile,
    remoteip: ctx.request.headers.get('cf-connecting-ip') ?? undefined,
  }),
}).then((r) => r.json());
if (!verify.success) throw new ActionError({ code: 'FORBIDDEN', message: 'verification failed' });
```

Semantics that bite: tokens are SINGLE-USE and valid 300 s (a resubmit needs a fresh token — reset the widget after each attempt); free tier is unlimited verifications, up to 20 widgets, 10 hostnames per widget. The secret lives as a Workers secret (via the secrets manager), never in the repo. For fixtures and CI, Cloudflare documents dummy keys (always-pass sitekey `1x00000000000000000000AA`, always-pass secret `1x0000000000000000000000000000000AA` — re-verify on the Turnstile testing docs page before relying on them).

## Storage — Supabase insert-only

Least-privilege lock: the Action inserts with the PUBLISHABLE key against an insert-only RLS policy — a leaked key yields write-only access to one table, nothing readable.

```sql
create table public.leads (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz not null default now(),
  name text not null, contact text not null, message text,
  consent boolean not null, consent_at timestamptz not null default now(),
  source text not null default 'form'
);
alter table public.leads enable row level security;
create policy leads_insert on public.leads for insert to anon with check (true);
grant insert on public.leads to anon;
-- NO select/update/delete policies: submitters write, nothing reads with that key.
```

Reading leads happens elsewhere (dashboard, secret-key job, eventual CRM sync) — never from the site. Keys are the NEW Supabase model (`sb_publishable_`/`sb_secret_`; legacy anon dies end of 2026); the key travels on the `apikey` header.

## Email notification — the native slot

The slot is Cloudflare Email Service (public beta since 2026-04-16), which wins on native-first FOR THIS EXACT CASE: sending to a VERIFIED DESTINATION ADDRESS in the account — the owner's own inbox — is free on ALL plans, including Workers Free, with the FROM on the site's domain.

Wiring — TWO prerequisites, both owner-side, then the binding (field-burned 2026-08-18):

1. Verify the owner's inbox (Email Routing destination-address flow, account level).
2. Onboard the FROM domain to Email Sending (`wrangler email sending enable <domain>` or dashboard). This creates a SENDING SUBDOMAIN with its own SPF/DKIM records — it does NOT touch the apex MX/SPF, so it coexists with an external mail provider (Google Workspace on the same zone, proven live). Destination verification alone is NOT enough — an un-onboarded from-domain fails the send with `E_SENDER_NOT_VERIFIED`.
3. Binding: `"send_email": [{ "name": "EMAIL" }]` in `wrangler.jsonc`; the handler calls `env.EMAIL.send({ to, from: { email, name }, subject, text, html })` — a plain object, and the Workers binding uses `email` in the from object (the REST API uses `address`; do not mix them). Errors throw with `E_*` codes — keep the send inside try/catch so a failure never fails the submission.

Beta caveats, stated honestly: no SLA, young deliverability reputation, conservative initial quotas. Field note (first live send, 2026-08-18): the notification DELIVERED to the verified Gmail-hosted inbox but landed in SPAM — expected for a fresh sending subdomain with zero reputation; the owner marking it not-spam trains the inbox, and the lead row in Supabase is the system of record either way — email is a courtesy channel, and a send failure must not fail the submission.

Documented fallback — Resend (`resend@6.20.0`, official Workers guide, 3,000 emails/mo free to ARBITRARY recipients): switch the slot when the brief needs auto-replies or confirmations to the LEAD's address on a $0 budget (arbitrary recipients on Email Service require Workers Paid), or when beta status is unacceptable. The slot is one function; swapping it touches nothing else.

## LFPDPPP consent (Mexico)

- The consent checkbox is REQUIRED and unchecked by default; its label links to the aviso de privacidad; the boolean and its timestamp are stored with the lead (the schema above). No pre-checked boxes, no consent buried in prose.
- The aviso itself is legal content — gated to the owner/lawyer, slot marked visibly until real (the catalog's placeholder discipline).
- If Turnstile runs in INVISIBLE mode, Cloudflare requires referencing the Turnstile Privacy Addendum in your privacy policy — a real line item for the aviso; managed/interaction-only mode avoids the requirement.

## The WhatsApp knob

In MX briefs WhatsApp is often the primary lead channel. It is not a competing architecture — it is a page-level knob that coexists with the form: a `wa.me` deep link CTA (plain anchor, zero CSP cost, zero JS). To keep the lead funnel measurable, log the click as a lead event — the same Action with `source: 'whatsapp'` and no message body, or the analytics layer once it exists (W3). Brief decides form-first vs WhatsApp-first placement; shipping BOTH with the form as the structured channel is the default.

## Abuse controls beyond Turnstile

- Honeypot field (`website`, visually hidden, any value rejects) — free, catches dumb bots that never execute Turnstile.
- Rate limiting when volume justifies it: the Workers `ratelimit` binding (GA since 2025-09; per-IP keys inside the same Worker — confirm current pricing before wiring) or ONE free WAF rate rule (10 s window, IP-based) on the `/_actions/` path. Neither is default; Turnstile + honeypot carries a brochure site.

## Gate amendments (CSP + budgets)

Composing this recipe amends two gates CONSCIOUSLY — the amendment is part of the recipe, not laxity:

- CSP: add `https://challenges.cloudflare.com` to script-src resources AND a `frame-src https://challenges.cloudflare.com` directive (the widget runs in an iframe). `connect-src 'self'` already covers pre-clearance. Hashes cannot cover an external script — these are origin allowances, the CSP-cheapest captcha there is.
- Perf budget: `resource-summary:third-party:count` moves from 0 to the widget's real count — a DOCUMENTED edit with this recipe as the reason; the api.js script loads async/deferred off the critical path. Any other third-party appearing later still fails the gate.

## Knobs per brief

| Knob | Default | Alternative and when |
|---|---|---|
| Turnstile mode | Managed, interaction-only | Invisible (adds the privacy-addendum line to the aviso) |
| Channel priority | Form + WhatsApp link, form-first | WhatsApp-first when the client operates in-chat |
| Email slot | Email Service to owner's verified inbox | Resend when auto-replies to leads are needed on $0 |
| Persistence | Supabase leads table (system of record) | Email-only for throwaway landings — accepts losing the record |
| Rate limiting | none (Turnstile + honeypot) | ratelimit binding or WAF rule on real abuse |

## Flip conditions (verbatim, standing)

Any of these promotes this recipe to a full `forms-lead-system` skill — re-scope, do not patch:

1. Uploads/multi-step/citas entran a los briefs (R2 presigned + Cal.com multiplican arquitecturas reales).
2. La ingesta a Twenty CRM entra in-scope ya.
3. Cloudflare Email Service supera a Resend y parte el slot de email — RESOLVED 2026-08-17: Email Service won the slot for owner-notification (free, native); the slot stays a knob, not a matrix — condition retired unless the slot fragments further.
4. Mixed prerendered+on-demand resulta frágil en Workers Builds en la práctica.

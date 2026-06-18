---
title: Cloudflare OAuth Worker, the COOP fix, and the /admin CSP
summary: Deploy the sveltia-cms-auth Worker, register a GitHub OAuth App, choose OAuth versus a personal access token, fix the COOP login failure, and write the hardened CSP and headers scoped to /admin.
last_updated: 2026-06-18
applies_to: sveltia-cms-auth on Cloudflare Workers, GitHub OAuth App, Workers Static Assets _headers
---

# Cloudflare OAuth Worker, the COOP fix, and the /admin CSP

> A non-technical client signs in with one click through a GitHub OAuth App fronted by the free sveltia-cms-auth Worker. The login is a popup that returns the token via postMessage, so the /admin block in _headers must relax COOP and carry a CSP that allowlists exactly what Sveltia talks to.

## Contents
- OAuth Worker versus personal access token
- Deploy sveltia-cms-auth
- Register the GitHub OAuth App
- The COOP login failure
- The /admin CSP and headers block
- Gotchas

## OAuth Worker versus personal access token

A personal access token needs no Worker and no OAuth App: a sole technical admin clicks Sign in with Token and pastes a token. It is the simplest path but it asks each editor to generate and manage a token, which a non-technical client will not do.

The OAuth Worker gives a one-click Sign in with GitHub. Use it for any non-technical editor. It is a small Cloudflare Worker on the free tier; Sveltia does not host it as a service because of the liability of a shared OAuth client, so each site deploys its own.

## Deploy sveltia-cms-auth

Deploy [sveltia/sveltia-cms-auth](https://github.com/sveltia/sveltia-cms-auth): the official Worker that completes the GitHub OAuth handshake. After deploy the URL is https://sveltia-cms-auth.SUBDOMAIN.workers.dev, shown in the Cloudflare Workers dashboard. Set these Worker environment variables:

| Variable | Value |
|---|---|
| GITHUB_CLIENT_ID | from the GitHub OAuth App |
| GITHUB_CLIENT_SECRET | from the GitHub OAuth App |
| ALLOWED_DOMAINS | the site hostname, for example www.example.com; comma-separated list and a wildcard like *.example.com are allowed |
| GITHUB_HOSTNAME | only for GitHub Enterprise Server; default github.com |

ALLOWED_DOMAINS is the security gate: the Worker rejects the handshake unless the calling page's origin matches an entry.

## Register the GitHub OAuth App

Register a new OAuth App on GitHub with the Authorization callback URL set to the Worker URL plus /callback, for example https://sveltia-cms-auth.SUBDOMAIN.workers.dev/callback. Copy the Client ID and generate a Client Secret into the Worker variables above. Then set base_url in config.yml to the Worker URL (not the callback). On login, Sveltia opens a popup to the Worker /auth endpoint, the Worker runs the GitHub handshake, and it posts the token back to the opener.

## The COOP login failure

The popup posts the token back with window.opener.postMessage. If the /admin document carries Cross-Origin-Opener-Policy same-origin, the browser severs the opener relationship, window.opener is null in the popup, the postMessage never arrives, and login hangs with no error. The site-wide default from web-security-headers is same-origin, so without an override the client cannot log in.

The fix is Cross-Origin-Opener-Policy same-origin-allow-popups scoped to /admin. It still isolates the document from cross-origin openers but preserves the relationship with popups the page itself opened, which is exactly the OAuth case. Do not use COEP require-corp on /admin. Cross-Origin-Resource-Policy does not break the popup; leave it as is.

## The /admin CSP and headers block

Because /admin is a static passthrough file, Astro's CSP meta element never lands on it, and the project _headers carries no site-wide CSP, so /admin has no CSP by default. Add a dedicated block in public/_headers, placed before the site-wide block so the more specific match applies, and harden it. Workers Static Assets supports _headers natively.

```
/admin/*
  Cross-Origin-Opener-Policy: same-origin-allow-popups
  Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com; img-src 'self' blob: data: https://avatars.githubusercontent.com https://raw.githubusercontent.com https://pub-HASH.r2.dev; media-src blob:; frame-src blob:; connect-src 'self' blob: data: https://api.github.com https://www.githubstatus.com https://ACCOUNT.JURISDICTION.r2.cloudflarestorage.com https://translation.googleapis.com
```

Directive notes, from the official Sveltia security guidance:

- script-src needs neither unsafe-eval nor unsafe-inline. style-src does need unsafe-inline, plus fonts.googleapis.com; font-src needs fonts.gstatic.com.
- img-src needs blob: and data:; media-src and frame-src need blob:.
- GitHub origins are api.github.com and www.githubstatus.com in connect-src, and avatars.githubusercontent.com and raw.githubusercontent.com in img-src. It is not github.com and not a githubusercontent wildcard.
- The OAuth Worker origin does not go in connect-src or frame-src; the popup is a top-level navigation, not a fetch or an iframe.
- Add the R2 endpoint and the R2 public URL only if media is on R2 (see media-r2.md), and the translation endpoint only for the service in use (translation.googleapis.com for Google Cloud Translation; see i18n-and-publishing.md).
- If the bundle is loaded from the CDN instead of npm, add https://unpkg.com to script-src and connect-src.

## Gotchas

- Login hangs silently means COOP. Check the /admin COOP value first.
- The callback URL must be the Worker URL plus /callback exactly; a mismatch shows as an OAuth redirect error from GitHub.
- base_url is the Worker root, not the /callback path.
- The CSP builder on the Sveltia site computes origins client-side; re-run it at build time to catch any new origin a future version adds.

## References
- [sveltia/sveltia-cms-auth](https://github.com/sveltia/sveltia-cms-auth): the Worker, environment variables, and OAuth App steps.
- [GitHub Backend | Sveltia CMS](https://sveltiacms.app/en/docs/backends/github): backend auth options, including the personal access token path.
- [Security | Sveltia CMS](https://sveltiacms.app/en/docs/security): the required CSP directives and per-backend origins.

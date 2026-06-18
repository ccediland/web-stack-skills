---
title: Media — repo-versus-R2 threshold and R2 setup
summary: Decide whether media commits to the repo or to Cloudflare R2, then configure the R2 bucket, token, CORS, and public URL for direct browser uploads.
last_updated: 2026-06-18
applies_to: Sveltia CMS GitHub backend with Cloudflare R2 media, Astro 6 on Cloudflare
---

# Media — repo-versus-R2 threshold and R2 setup

> Commit media to the repo for small or mostly-static sites, where Sveltia's client-side WebP conversion keeps files tiny. Move to Cloudflare R2 above the threshold, or whenever media is large, because the GitHub backend has no Git LFS. R2 uploads go browser-to-R2 directly with no proxy.

## Contents
- The threshold rule
- Why no LFS forces R2 for large media
- R2 configuration
- CORS, which is required
- The public URL
- CSP additions
- Gotchas

## The threshold rule

Default to committing media to the repo. Sveltia converts images to WebP and resizes them client-side before commit, so a typical optimized image is well under a megabyte, and a marketing or blog site stays far below any practical repo limit. Repo media has zero extra infrastructure and moves with the content if the site is ever migrated.

Move to R2 when any of these holds:

| Condition | Why R2 |
|---|---|
| Media is large or frequent (galleries, video, large PDFs) | repo bloat is permanent in git history and slows every clone and build |
| An upload exceeds GitHub's browser file size cap | the browser path cannot push it at all |
| Total media pushes the repo toward a gigabyte | clone and CI times degrade |

GitHub warns at 50 MiB, blocks files over 100 MiB, and the browser upload path caps lower than the API path. There is no hard repo size limit, but a repo over a gigabyte is a maintenance problem.

## Why no LFS forces R2 for large media

The Sveltia GitHub backend does not support Git LFS, an API limitation. So there is no large-file escape hatch inside the repo: anything beyond GitHub's per-file limits has to live in R2. This is the main reason R2 exists in this recipe rather than being optional polish.

## R2 configuration

R2 is S3-compatible with zero egress fees. Sveltia uploads browser-to-R2 directly with AWS Signature Version 4, so there is no backend proxy. Create a bucket and an R2 API token with Object Read and Write permission. Configure the media library in config.yml:

```yaml
media_libraries:
  cloudflare_r2:
    access_key_id: <64-hex-access-key-id>
    bucket: my-bucket
    account_id: <account-id>
    public_url: https://pub-HASH.r2.dev
    prefix: cms-uploads/        # optional
    jurisdiction: default       # optional: default | eu | fedramp
```

The access_key_id goes in config; the matching Secret Access Key is typed by the user in the CMS UI the first time they open the media library and is never stored in config.

## CORS, which is required

The SigV4 upload sends custom headers, which triggers a CORS preflight, so the bucket must allow the CMS origin. Configure it under R2 bucket Settings, CORS Policy:

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "PUT", "HEAD"],
    "AllowedOrigins": ["https://your-cms-domain.com"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

## The public URL

public_url is how saved media is previewed and served. The pub-HASH.r2.dev development URL is rate-limited and meant for development; for production, connect a custom domain to the bucket and use that as public_url. Asset URLs are public_url plus the object key.

## CSP additions

When media is on R2, add to the /admin CSP block (see cloudflare-auth-worker.md):

- connect-src: https://ACCOUNT.JURISDICTION.r2.cloudflarestorage.com (the S3 API endpoint the upload hits).
- img-src: the public_url origin (https://pub-HASH.r2.dev or the custom domain).

Repo media needs no CSP change beyond the default img-src 'self'.

## Gotchas

- Skipping the CORS policy makes uploads fail with an opaque preflight error. It is required, not optional.
- The r2.dev public URL is throttled; do not use it in production.
- jurisdiction must match where the bucket was created (default, eu, or fedramp); a mismatch makes the endpoint host wrong.
- The Secret Access Key is per-user UI state, not config. A new editor on a new machine re-enters it.

## References
- [Cloudflare R2 Integration | Sveltia CMS](https://sveltiacms.app/en/docs/media/cloudflare-r2): the config block, CORS, token, and public URL.
- [About large files on GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github): the 50 MiB warning, 100 MiB block, and browser cap.

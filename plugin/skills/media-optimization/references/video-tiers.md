---
title: Video tiers — public/, R2, Media Transformations, and the Stream flip
summary: The compression recipe and markup for self-hosted mp4 under the 25 MiB Static Assets cap, the R2 custom-domain escalation, Media Transformations derivatives with verified billing semantics, reduced-motion and poster wiring, and when Stream is honest.
last_updated: 2026-08-17
applies_to: Cloudflare Workers Static Assets (25 MiB/file) · R2 public buckets · Media Transformations (GA) · Stream
---

# Video tiers

> Astro has NO native video support — video on this stack is a plain `video` element plus a hosting decision. The decision is a ladder: each tier is forced by a verified platform constraint, not preference.

## Contents

- The ladder
- Tier 0 — mp4 in public/
- Markup — poster, preload, reduced motion
- Tier 1 — R2 behind a custom domain
- Tier 2 — Media Transformations derivatives
- The Stream flip
- Billing semantics worth memorizing

## The ladder

| Tier | Home | Forced by | Cost |
|---|---|---|---|
| 0 | `public/` on Static Assets | File ≤ 25 MiB (hard per-file cap, free and paid plans) | Zero — asset requests are free and unlimited |
| 1 | R2 public bucket behind a CUSTOM domain | File over 25 MiB, or video count bloating the asset manifest | R2 storage (generous free tier); custom domain gets Cloudflare cache/WAF — `r2.dev` is rate-limited and non-production |
| 2 | Media Transformations over the R2 original | Needing multiple sizes/trims without storing N encodes | Shares the 5,000/month free unique-operations pool with image transforms; then $0.50/1k uniques; video output bills 1 op PER SECOND of output |
| flip | Stream | Long-form, adaptive bitrate, or live | $5/1k minutes stored + $1/1k minutes delivered; NO free tier |

## Tier 0 — mp4 in public/

A decorative loop or short promo belongs here, compressed far below the cap:

    ffmpeg -i raw.mov -an -c:v libx264 -crf 28 -preset slow \
      -vf "scale=1280:-2" -movflags +faststart hero-loop.mp4

- H.264 + AAC (or `-an` for no audio on decorative loops) is the universal baseline; `-movflags +faststart` moves the moov atom up so playback starts before the download completes.
- `-crf 28` on a short loop lands single-digit MiB at 720p; budget a decorative loop in the low single digits of MiB — 25 MiB is the platform's ceiling, never the target.
- An optional WebM/AV1 sibling `source` saves ~30% for supporting browsers; on a short loop the encode complexity usually is not worth it — measure first.

## Markup — poster, preload, reduced motion

```astro
---
import { getImage } from 'astro:assets';
import posterSrc from '../assets/hero-poster.jpg';
const poster = await getImage({ src: posterSrc, format: 'avif', width: 1280 });
---
<video muted loop playsinline preload="metadata" poster={poster.src} data-autoplay>
  <source src="/hero-loop.mp4" type="video/mp4" />
</video>

<script>
  const reduced = window.matchMedia('(prefers-reduced-motion: reduce)');
  for (const v of document.querySelectorAll('video[data-autoplay]')) {
    if (!reduced.matches) v.play();
  }
</script>
```

- The poster goes through the IMAGE pipeline (AVIF via `getImage`) — first paint is an optimized still, not a video frame.
- `preload="metadata"` keeps the initial payload to headers + moov; the file streams when it plays.
- Autoplay is gated behind `prefers-reduced-motion` in SCRIPT (not just CSS): a reduced-motion visitor gets the poster, full stop. This composes with the catalog's standing reduced-motion posture (motion-system owns the per-engine rules; this is the video instance of the same principle). The script is bundled, so the site's hash-based CSP covers it automatically.
- No `autoplay` attribute: autoplay is a JS decision so the gate cannot be bypassed by markup.
- Content video (talking-head, tutorials) gets `controls`, captions via `<track>` (an a11y-deep concern), and NO autoplay.

## Tier 1 — R2 behind a custom domain

- Create the bucket, enable public access, and attach a CUSTOM domain on the zone — that routes traffic through Cloudflare's cache and WAF. The `r2.dev` development URL is rate-limited and explicitly not for production.
- The `video` markup does not change; only the `src` origin does. Add the origin to the site's CSP `media-src` (or `default-src` fallback list) — the composition seam every off-origin asset pays.
- Range requests are standard S3-style GETs on R2; progressive mp4 with faststart plays fine.

## Tier 2 — Media Transformations derivatives

One R2 original, sized variants minted at the edge:

    https://<zone-host>/cdn-cgi/media/mode=video,width=640,time=0s,duration=10s/<source-url>

- Modes: `video` (optimized mp4 out), `frame` (a still — poster without an encode step), `spritesheet`, `audio`. Input caps: ≤100 MB, ≤10 min, H.264 mp4.
- Same-zone origin restriction by default — the source URL must live on the same zone (the R2 custom domain satisfies this).
- There is also a Workers binding for Media Transformations billed PER OPERATION (not per unique) — avoid it on hot paths; the URL form bills per unique.

## The Stream flip

When the brief is real video content at scale — long-form, adaptive bitrate (HLS/DASH), live — stop assembling tiers and buy Stream: managed encoding (free ingress), the `<stream>` player or HLS URL out. It has NO free tier ($5/1k min stored, $1/1k min delivered), which is exactly why it is the flip and not the ladder: paying it must be a product decision.

## Billing semantics worth memorizing

- The 5,000 free unique operations per month are ONE pool shared by Image Transformations and Media Transformations on the zone.
- A video derivative bills 1 operation per SECOND of output — a 20-second variant = 20 uniques; a poster `frame` = 1. Sized video variants consume the free pool fast; mint few, cache forever.
- Uniques are counted per distinct transformation URL over the billing window; the free pool failing means ERRORS on new transformations (fail-closed, like Images) — a site relying on tier 2 for its hero has a hard availability dependency on the quota. Decorative video should always have a tier-0 fallback file.

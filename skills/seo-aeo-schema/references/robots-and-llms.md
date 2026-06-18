---
title: robots.txt and llms.txt
summary: Verified AI-crawler tokens, the access-versus-appearance rule, the baseline robots.txt, the honest llms.txt framing, and the shared prerendered text-endpoint pattern.
last_updated: 2026-06-17
applies_to: astro@6.4.7, @astrojs/cloudflare@13.7.0
---

# robots.txt and llms.txt

## Table of contents

- AI-crawler tokens
- Access is not appearance
- The baseline robots.txt
- llms.txt: what it is and is not
- The prerendered text-endpoint pattern
- Cloudflare caveats

## AI-crawler tokens

robots.txt is the enforceable surface for crawler access. Reputable bots honor it; aggressive scrapers ignore it. Verified vendor tokens as of June 2026:

| Token | Vendor | Purpose |
|---|---|---|
| Googlebot | Google | Search crawl and index, must stay allowed for AI Overviews |
| Google-Extended | Google | Robots token, opts out of Gemini training and grounding, not a crawler |
| Bingbot | Microsoft | Bing index, also feeds Copilot |
| GPTBot | OpenAI | Model training |
| OAI-SearchBot | OpenAI | ChatGPT search indexing and citations |
| ChatGPT-User | OpenAI | User-initiated fetch |
| ClaudeBot | Anthropic | Model training |
| Claude-SearchBot | Anthropic | Search and retrieval indexing |
| Claude-User | Anthropic | User-initiated fetch |
| PerplexityBot | Perplexity | Index for citations |
| Perplexity-User | Perplexity | User-initiated fetch, may not obey robots.txt |
| CCBot | Common Crawl | Open corpus that feeds many training sets |
| Meta-ExternalAgent | Meta | Training and indexing |
| Applebot | Apple | Siri and Spotlight search |
| Applebot-Extended | Apple | Robots token, opts out of Apple Intelligence training |

Do not block `facebookexternalhit`; it renders link previews and is not an AI crawler.

## Access is not appearance

Three rules that the SEO blogs routinely conflate:

- Disallowing a crawler is not the same as keeping a page out of AI answers. To keep content out entirely, use `noindex`, authentication, or `nosnippet`. A disallowed page can still surface as a title and link if a third party returns it.
- Googlebot must be allowed for Google's AI features to use the content. Google-Extended governs only training and grounding; it does not remove a site from AI Overviews and does not affect ranking.
- robots.txt controls access. llms.txt does not; it is not an access-control mechanism and blocks nothing.

## The baseline robots.txt

Allow search and AI-search crawlers so the site stays visible and citable. AI-training opt-out is a deliberate choice the implementer makes, shown here commented out.

```
# Search and AI-search crawlers: allow for visibility and citations
User-agent: Googlebot
Allow: /
User-agent: Bingbot
Allow: /
User-agent: Applebot
Allow: /
User-agent: OAI-SearchBot
Allow: /
User-agent: Claude-SearchBot
Allow: /
User-agent: PerplexityBot
Allow: /

# AI training crawlers: opt out by uncommenting
# User-agent: GPTBot
# Disallow: /
# User-agent: ClaudeBot
# Disallow: /
# User-agent: CCBot
# Disallow: /
# User-agent: Google-Extended
# Disallow: /
# User-agent: Applebot-Extended
# Disallow: /
# User-agent: Meta-ExternalAgent
# Disallow: /

User-agent: *
Allow: /

Sitemap: https://example.com/sitemap-index.xml
```

## llms.txt: what it is and is not

llms.txt is a proposed convention for a curated Markdown index of a site's content, with an H1 name, a blockquote summary, and H2 sections of links. A companion `llms-full.txt` inlines the entire documentation as one Markdown file. The two have different jobs: index and navigation versus a full-context dump.

The honest framing: no major AI provider consumes llms.txt as a production ranking or answer signal. Google has stated on the record that it does not support it, and named-methodology studies find that AI crawlers fetch it negligibly. As an SEO lever it does nothing.

Where it is legitimate: as a Business-to-Agent and agent-readiness surface. AI coding agents and documentation platforms use it as a routing layer, several large vendors ship one, and Lighthouse 13.3 (released 7 May 2026) added an agentic-browsing audit that checks for it. Ship a minimal llms.txt for that positioning and for the Lighthouse check, and frame it as such, not as SEO.

## The prerendered text-endpoint pattern

Both robots.txt and llms.txt are emitted the same way. Because the Cloudflare adapter defaults to on-demand rendering, a text endpoint must opt into prerendering with `export const prerender = true`, or it renders in the Worker instead of becoming a static file in `dist/client`.

```ts
// src/pages/robots.txt.ts
import type { APIRoute } from 'astro';
export const prerender = true;

export const GET: APIRoute = ({ site }) => {
  const sitemap = new URL('sitemap-index.xml', site).href;
  const body = [
    'User-agent: Googlebot',
    'Allow: /',
    '',
    'User-agent: *',
    'Allow: /',
    '',
    `Sitemap: ${sitemap}`,
    '',
  ].join('\n');
  return new Response(body, { headers: { 'Content-Type': 'text/plain; charset=utf-8' } });
};
```

```ts
// src/pages/llms.txt.ts
import type { APIRoute } from 'astro';
import { getCollection } from 'astro:content';
export const prerender = true;

export const GET: APIRoute = async ({ site }) => {
  const docs = await getCollection('docs');
  const links = docs
    .map((d) => `- [${d.data.title}](${new URL(`${d.id}/`, site).href})`)
    .join('\n');
  const body = `# Example\n\n> Premium brand site.\n\n## Pages\n${links}\n`;
  return new Response(body, { headers: { 'Content-Type': 'text/plain; charset=utf-8' } });
};
```

A prerendered endpoint earns its place over a static `public/` file when the content is generated from collections, as llms.txt is. A fixed robots.txt can live in `public/robots.txt` instead; the endpoint form is shown for parity and for when the sitemap URL should derive from `site`.

## Cloudflare caveats

Add a `.assetsignore` in `public/` per the adapter setup. The adapter serves the build output from `dist/client`; confirm routing and `.assetsignore` do not shadow `robots.txt`, `llms.txt`, or the sitemap files. Header control for static assets is a `public/_headers` file, since the v13 adapter emits no static headers; this is the web-security-headers skill's territory, referenced here only so the files are not double-managed.

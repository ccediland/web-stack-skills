---
title: Validation and AEO framing
summary: Validation tooling, the exported CI script and its handoff to perf-ci-gates, the 2026 rich-result types, and what JSON-LD does and does not do for AI surfaces.
last_updated: 2026-06-17
applies_to: schema.org, Google Search Central, June 2026
---

# Validation and AEO framing

## Validation tooling

Two complementary tools, neither of which is a build step on its own:

- Rich Results Test (Google) validates only Google rich-result-eligible types and previews the rich result. It loses FAQ support in June 2026.
- Schema Markup Validator at validator.schema.org validates syntax and compliance against the full Schema.org vocabulary. It does not report Google rich-result eligibility.

Run both during authoring. Because schema-dts does not validate @id cross-references (see `json-ld-graph.md`), these runtime checks are the real test of whether the graph is wired correctly.

## The exported CI script

For automation, this skill exports a small script that extracts every `application/ld+json` block from built HTML, parses each as JSON, and checks required properties per type. It depends on no third-party validator, which keeps maintenance low; avoid the older structured-data-testing-tool package unless its current maintenance has been confirmed.

```js
// scripts/check-jsonld.mjs — run against the built output
import { readFileSync, readdirSync } from 'node:fs';
import { join } from 'node:path';

const REQUIRED = {
  Organization: ['name', 'url'],
  WebSite: ['url'],
  WebPage: ['url'],
  BreadcrumbList: ['itemListElement'],
  Article: ['headline', 'author'],
};

function* htmlFiles(dir) {
  for (const e of readdirSync(dir, { withFileTypes: true })) {
    const p = join(dir, e.name);
    if (e.isDirectory()) yield* htmlFiles(p);
    else if (e.name.endsWith('.html')) yield p;
  }
}

let failed = 0;
for (const file of htmlFiles('dist/client')) {
  const html = readFileSync(file, 'utf8');
  const blocks = [...html.matchAll(/<script type="application\/ld\+json"[^>]*>([\s\S]*?)<\/script>/g)];
  for (const [, raw] of blocks) {
    let data;
    try { data = JSON.parse(raw); }
    catch { console.error(`${file}: invalid JSON-LD`); failed++; continue; }
    const nodes = data['@graph'] ?? [data];
    for (const node of nodes) {
      const need = REQUIRED[node['@type']];
      if (need) for (const k of need) if (!(k in node)) {
        console.error(`${file}: ${node['@type']} missing ${k}`);
        failed++;
      }
    }
  }
}
process.exit(failed ? 1 : 0);
```

The merge gate is not built here. The perf-ci-gates skill owns the GitHub Actions workflow; it adds a step that runs this script against `dist/client` after the build. This skill authors the structured data and the check; that skill decides when the check blocks a merge.

## 2026 rich-result types

What still earns a Google rich result, and what no longer does, as of June 2026:

| Type | Status |
|---|---|
| Product, Review, AggregateRating | Active |
| Article | Active |
| Recipe | Active |
| Video | Active |
| Organization | Active |
| LocalBusiness | Active |
| BreadcrumbList | Active |
| Event | Active |
| FAQPage | Rich result deprecated 7 May 2026; type still valid and still parsed |
| HowTo | Fully deprecated |
| Book Actions, Course Info, Claim Review, Estimated Salary, Learning Video, Special Announcement, Vehicle Listing | Retired June 2025 |

FAQ specifics: rich results stopped showing 7 May 2026, Rich Results Test and Search Console reporting end June 2026, and the Search Console API ends August 2026. FAQPage remains a valid type, Google still parses it, and rankings are unaffected. Do not add FAQPage as a rich-result play; keep it only where genuine question-and-answer content exists.

Choose FAQPage versus QAPage correctly: FAQPage is for the site owner's single official answer; QAPage is for a page where users submit multiple answers, such as a forum thread. The wrong choice gets the markup ignored.

## What JSON-LD does and does not do for AI

The framing that keeps this skill honest. Google's own AI-features guidance states there are no additional requirements to appear in AI Overviews or AI Mode, no special schema.org structured data to add, and no new machine-readable files to create. Structured data does, however, have to match visible page content, or it risks a manual action.

So the value of JSON-LD is rich-result eligibility plus entity clarity for entity recognition, in which Organization and Person are central. It does not cause inclusion in AI Overviews, AI Mode, or chatbot citations; AI surfaces extract from the visible text, which the schema must mirror.

The levers that actually move AI visibility are content-side, not markup: clear, quotable content with citations and statistics raises a source's odds of being cited, and the large majority of AI citations come from brand-controlled surfaces. Author schema for what the page is, keep it matched to visible content, and put the real effort into the content the AI reads.

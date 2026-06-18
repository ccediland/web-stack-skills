---
title: Typed JSON-LD @graph with schema-dts 2.0.0
summary: schema-dts 2.0.0 types, the canonical cross-@id entity graph, the Astro render component, the @id non-validation limitation, and why inline JSON-LD is exempt from Astro's hash-based CSP.
last_updated: 2026-06-17
applies_to: schema-dts@2.0.0, astro@6.4.7, @astrojs/cloudflare@13.7.0
---

# Typed JSON-LD @graph with schema-dts 2.0.0

## Table of contents

- The package and what it buys
- The @id non-validation limitation
- The canonical cross-@id graph
- The typed graph builder
- The Astro render component
- Why JSON-LD is CSP-exempt
- Pin and review-gate

## The package and what it buys

schema-dts is a set of TypeScript typings for the Schema.org vocabulary in JSON-LD. It is a devDependency with zero runtime footprint: it ships types only and emits nothing into the bundle. Version 2.0.0 tracks Schema.org v30.

```bash
npm i -D schema-dts
```

```ts
import type { Graph, WithContext, Organization, Article } from 'schema-dts';
```

Use `WithContext<T>` for a single top-level object and `Graph` for a multi-node graph. In 2.0.0 `WithContext<T>` is intersection-only (`T & { "@context": "https://schema.org" }`) and can no longer be a graph, so import `Graph` explicitly when building a graph. The `Graph` interface is `{ "@context": "https://schema.org"; "@graph": readonly Thing[] }`.

What the types buy: per-node property names and value types are checked against the vocabulary, which catches typos and wrong-typed values and gives editor autocomplete over 800-plus types. What they do not buy is in the next section.

## The @id non-validation limitation

This is the load-bearing caveat. schema-dts types `@id` as a bare string (`IdReference = { "@id": string }`). It performs zero referential validation: it cannot verify that a referenced `@id` exists elsewhere in the graph, and it cannot verify that the referenced node is the right type. Object-valued properties are generated as `SchemaValue<T | IdReference>`, so any string passes the `IdReference` arm of the union.

Consequence: a cross-reference with a typo in the `@id`, or one pointing at the wrong entity type, compiles cleanly and silently produces a broken graph. The dependency guarantees correct nodes, not a correct graph. Validate the rendered JSON-LD with the Rich Results Test and validator.schema.org, and treat that as the real check on graph wiring. See `validation-and-aeo.md`.

## The canonical cross-@id graph

One centralized graph per page, each node carrying a stable `@id` and linking to the others by `@id`. This is the rendered target for a premium brand or content site.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    { "@type": "Organization", "@id": "https://example.com/#org",
      "name": "Example", "url": "https://example.com/",
      "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" } },
    { "@type": "WebSite", "@id": "https://example.com/#website",
      "url": "https://example.com/", "name": "Example",
      "publisher": { "@id": "https://example.com/#org" } },
    { "@type": "WebPage", "@id": "https://example.com/page/#webpage",
      "url": "https://example.com/page/", "name": "Page title",
      "isPartOf": { "@id": "https://example.com/#website" },
      "breadcrumb": { "@id": "https://example.com/page/#breadcrumb" } },
    { "@type": "BreadcrumbList", "@id": "https://example.com/page/#breadcrumb",
      "itemListElement": [
        { "@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com/" },
        { "@type": "ListItem", "position": 2, "name": "Page", "item": "https://example.com/page/" } ] },
    { "@type": "Article", "@id": "https://example.com/page/#article",
      "headline": "Page title",
      "isPartOf": { "@id": "https://example.com/page/#webpage" },
      "mainEntityOfPage": { "@id": "https://example.com/page/#webpage" },
      "author": { "@type": "Person", "name": "Author" },
      "publisher": { "@id": "https://example.com/#org" } }
  ]
}
```

The Article node is per-page and optional; swap it for Product, Recipe, Event, or another type that still earns rich results (see `validation-and-aeo.md`). Drop it on pages with no primary entity.

Sitelinks searchbox is a separate optional addition to the WebSite node. It uses a `query-input` property that is not part of the Schema.org vocabulary, so schema-dts rejects it and the node needs a type assertion. Google has largely retired the sitelinks searchbox, so omit it unless the site genuinely qualifies.

## The typed graph builder

A small builder keeps the @id wiring in one place. The cross-reference stubs are the strings schema-dts will not validate, so this is where care matters.

```ts
import type { Graph } from 'schema-dts';

export function buildGraph(o: {
  siteUrl: string; pageUrl: string; orgName: string; logoUrl: string;
  pageTitle: string; breadcrumbs: { name: string; url: string }[];
  article?: { headline: string; author: string; datePublished: string };
}): Graph {
  const orgId = `${o.siteUrl}#org`;
  const siteId = `${o.siteUrl}#website`;
  const pageId = `${o.pageUrl}#webpage`;
  const crumbId = `${o.pageUrl}#breadcrumb`;
  return {
    '@context': 'https://schema.org',
    '@graph': [
      { '@type': 'Organization', '@id': orgId, name: o.orgName, url: o.siteUrl,
        logo: { '@type': 'ImageObject', url: o.logoUrl } },
      { '@type': 'WebSite', '@id': siteId, url: o.siteUrl, name: o.orgName,
        publisher: { '@id': orgId } },
      { '@type': 'WebPage', '@id': pageId, url: o.pageUrl, name: o.pageTitle,
        isPartOf: { '@id': siteId }, breadcrumb: { '@id': crumbId } },
      { '@type': 'BreadcrumbList', '@id': crumbId,
        itemListElement: o.breadcrumbs.map((b, i) => ({
          '@type': 'ListItem', position: i + 1, name: b.name, item: b.url })) },
      ...(o.article ? [{
        '@type': 'Article', '@id': `${o.pageUrl}#article`,
        headline: o.article.headline, isPartOf: { '@id': pageId },
        mainEntityOfPage: { '@id': pageId },
        author: { '@type': 'Person', name: o.article.author },
        publisher: { '@id': orgId }, datePublished: o.article.datePublished,
      }] : []),
    ],
  } as Graph;
}
```

For a node that mixes two types (for example a Product that is also a SoftwareApplication), use the concrete leaf types rather than a union alias: `MergeLeafTypes<[ProductLeaf, SoftwareApplicationLeaf]>`. Large discriminated unions on `@type` can stress the compiler, so prefer leaf types over wide unions on hot paths.

## The Astro render component

Render the graph as a single inline data block in the base layout. Escape the characters that could close the script tag, which is the real risk for inline JSON-LD. Use `is:inline` so Astro leaves the block untouched.

```astro
---
import type { Graph } from 'schema-dts';
interface Props { graph: Graph }
const { graph } = Astro.props;
const json = JSON.stringify(graph)
  .replace(/</g, '\\u003C')
  .replace(/>/g, '\\u003E')
  .replace(/&/g, '\\u0026');
---
<script type="application/ld+json" is:inline set:html={json} />
```

Pass one `Graph` per page from the builder. One graph per page beats scattering per-component blocks, because cross-@id integrity only holds when the nodes live together.

## Why JSON-LD is CSP-exempt

A `<script type="application/ld+json">` is a data block in the HTML spec: the type is not a JavaScript type, so the browser never executes it. CSP `script-src` governs script execution, so it does not apply to a data block and the browser raises no violation for it.

Practical effect on this stack: Astro 6's hash-based CSP (the web-security-headers skill) hashes the executable scripts it bundles, such as client islands and `define:vars` blocks. It does not hash an author-written `ld+json` block, and it does not need to. Do not add an entry for the JSON-LD to `security.csp.scriptDirective.hashes`. The CI-versus-local hash drift caused by Vite chunk boundaries affects bundled inline scripts, not a static `ld+json` block, which carries no hash at all. Structured data ships cleanly under the strict meta-element CSP that Cloudflare delivers.

## Pin and review-gate

Pin `schema-dts@2.0.0` as a devDependency. Breaking changes from 1.x: Role became non-recursive, Quantity became a core DataType in Schema.org v30 so a few previously legal assignments now error, and the generator now depends on a separate `schema-dts-lib` package. Holding at 1.1.5 gains nothing and falls behind the vocabulary.

Review-gate: if the TypeScript compiler chokes on graph depth, move to `...Leaf` types and `MergeLeafTypes`, not down to 1.1.5.

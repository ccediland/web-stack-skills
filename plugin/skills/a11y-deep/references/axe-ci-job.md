---
title: The axe CI job — scan script, workflow wiring, and the zero-violations posture
summary: The full-page-sweep scan script over the built dist, the GitHub Actions job that runs it beside the Lighthouse gate, the tag policy with target-size enabled, exclusion discipline, and the anti-vacuous-gate habit.
last_updated: 2026-08-17
applies_to: '@axe-core/playwright@4.13.0 · playwright chromium · GitHub Actions · Astro dist/client output'
---

# The axe CI job

> One script, one job: build the site, serve the static output, run axe on EVERY built page, fail on any violation. The job is deliberately boring — the design decisions are the tag set, the enabled target-size rule, and the refusal to baseline.

## Contents

- The scan script
- The workflow job
- Tag policy
- Exclusion discipline
- The anti-vacuous-gate habit
- Scaling the sweep

## The scan script

```js
// scripts/axe-scan.mjs — zero-dep static server + full-page axe sweep
import { createServer } from 'node:http';
import { readFile, readdir } from 'node:fs/promises';
import { join, extname } from 'node:path';
import { chromium } from 'playwright';
import { AxeBuilder } from '@axe-core/playwright';

const DIST = './dist/client';
const EXCLUDE = [/^\/admin\//]; // app shells outside the audited set, same as LHCI

const TYPES = { '.html': 'text/html', '.css': 'text/css', '.js': 'text/javascript',
  '.svg': 'image/svg+xml', '.png': 'image/png', '.jpg': 'image/jpeg',
  '.avif': 'image/avif', '.webp': 'image/webp', '.woff2': 'font/woff2' };

const server = createServer(async (req, res) => {
  let path = decodeURIComponent(new URL(req.url, 'http://x').pathname);
  if (path.endsWith('/')) path += 'index.html';
  try {
    const body = await readFile(join(DIST, path));
    res.writeHead(200, { 'content-type': TYPES[extname(path)] ?? 'application/octet-stream' });
    res.end(body);
  } catch {
    res.writeHead(404); res.end();
  }
}).listen(4322);

const files = await readdir(DIST, { recursive: true });
const pages = files
  .filter((f) => f.endsWith('index.html'))
  .map((f) => `/${f.replace(/index\.html$/, '')}`)
  .filter((route) => !EXCLUDE.some((rx) => rx.test(route)));
if (pages.length === 0) { console.error('axe: 0 pages found — vacuous gate, failing'); process.exit(1); }

// AxeBuilder refuses pages from an implicit context ("Please use
// browser.newContext()") — create the context explicitly, then the page.
const browser = await chromium.launch();
const context = await browser.newContext();
const page = await context.newPage();
let total = 0;
for (const route of pages.sort()) {
  await page.goto(`http://localhost:4322${route}`, { waitUntil: 'load' });
  const results = await new AxeBuilder({ page })
    .withTags(['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa', 'wcag22aa'])
    .options({ rules: { 'target-size': { enabled: true } } })
    .analyze();
  total += results.violations.length;
  console.log(`axe ${route} — ${results.violations.length} violations`);
  for (const v of results.violations) {
    console.log(`  [${v.impact}] ${v.id}: ${v.help} (${v.nodes.length} nodes)`);
    for (const n of v.nodes.slice(0, 3)) console.log(`    ${n.target.join(' ')}`);
  }
}
await browser.close();
server.close();
console.log(`axe: scanned ${pages.length} pages, ${total} violations`);
process.exit(total > 0 ? 1 : 0);
```

Notes:

- The server is a ~15-line static file server over `dist/client` — no extra dependency, no workerd. axe audits DOM, not headers; static serving is sufficient and deterministic.
- `waitUntil: 'load'` (not networkidle) — pages with widgets or sockets never reach networkidle.
- Violations print with impact, rule id, and up to three target selectors — enough to fix from the CI log without re-running locally.

## The workflow job

```yaml
  axe:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build
        env:
          DATA_SOURCE: seed # keep CI standalone — same decision as the LHCI job
      - run: npx playwright install --with-deps chromium
      - run: node scripts/axe-scan.mjs
```

- Lives in the SAME workflow file as the Lighthouse and Biome gates — perf-ci-gates owns that file; this job is a11y-deep's contribution to it.
- Chromium comes from `playwright install` per run (or a cached toolchain if the repo already provisions one); the wrapper's peer dep is `playwright-core >= 1.0.0`, so no version-coupling to manage.
- devDependencies: `@axe-core/playwright@4.13.0` and `playwright` (exact-pinned like every load-bearing tool in the stack).

## Tag policy

- `wcag2a, wcag2aa, wcag21a, wcag21aa, wcag22aa` = the WCAG 2.2 AA claim surface. Best-practice rules (`best-practice` tag) are NOT in the gate: they are advisory, and gating on advice breeds exclusions.
- `target-size` is tagged `wcag22aa` but ships DISABLED in axe 4.13 — the explicit `.options({ rules: { 'target-size': { enabled: true } } })` line turns the only automated WCAG 2.2 rule on. Lighthouse already runs its own target-size audit, so enabling it here keeps the two instruments consistent.
- Review-gate: when axe flips target-size to enabled-by-default, the options line becomes redundant — harmless, but drop it on the bump that changes the default.

## Exclusion discipline

Zero violations is the posture; exclusions are the escape valve and they are NARROW:

- Per-rule + per-selector, in the scan script, each with a comment naming why and an owner decision reference (e.g. a third-party embed that cannot be fixed at source).
- NEVER a global baseline or "new violations only" ratchet — that machinery exists for adopting dirty codebases; greenfield sites on this stack start clean and stay clean.
- Route-level exclusion is reserved for app shells outside the audited set (the CMS admin), mirroring the LHCI collect list — the two gates must agree on what "the site" is.

## The anti-vacuous-gate habit

The script fails hard when it finds ZERO pages — a moved dist path or broken glob must never produce a green job that scanned nothing (the same failure mode the composition playbook documents for the Lighthouse assert step). When reading CI, confirm the `axe: scanned N pages` line and that N matches the built page count you expect.

## Scaling the sweep

Full sweep is the default: a static site scans in seconds per page and catches per-content-entry regressions samples miss. When a site's page count grows into the hundreds (content collections), switch to template exemplars + N random pages per run — the WCAG-EM sampling idea applied to the automated layer — and say so in the scan script comment.

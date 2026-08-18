---
title: The visual gate — spec, config, workflow job, and the baseline-update flow
summary: The exact Playwright config and spec for screenshotting built pages, the CI job that joins the other three gates, the CI-generated baseline update flow, and the flake discipline (masks, thresholds, animations).
last_updated: 2026-08-18
applies_to: "@playwright/test@1.62.1 (same pin as the axe job) · GitHub Actions ubuntu runners · Astro dist/client output"
---

# The visual gate

> One config, one spec, one job. Baselines are born in CI and live in git; a locally-rendered baseline is a bug, not a convenience.

## Contents

- The config
- The spec
- The workflow job
- The baseline-update flow (CI-generated)
- Flake discipline
- Reading a failure

## The config

```ts
// playwright.config.ts
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests',
  expect: {
    toHaveScreenshot: {
      // Explicit, conscious threshold — the unset default is exact-match,
      // which turns anti-aliasing noise into red builds.
      maxDiffPixelRatio: 0.01,
      // Forces CSS animations/transitions to their end state before capture.
      animations: 'disabled',
    },
  },
  webServer: {
    // Serve the BUILT site — the gate asserts what ships, not the dev server.
    command: 'python3 -m http.server 4173 -d dist/client',
    port: 4173,
    reuseExistingServer: false,
  },
  projects: [
    { name: 'desktop', use: { viewport: { width: 1280, height: 800 } } },
    { name: 'mobile', use: { viewport: { width: 390, height: 844 } } },
  ],
});
```

- `@playwright/test` shares its version with the `playwright` package the axe job already pins — install the SAME version and bump them together (regenerating baselines on every bump).
- The static python server is a zero-dependency stand-in for the asset layer; navigate with trailing slashes so directory indexes resolve.

## The spec

```ts
// tests/visual.spec.ts
import { expect, test } from '@playwright/test';

const PAGES = ['/', '/contacto/', '/galeria/'];

for (const path of PAGES) {
  test(`visual ${path}`, async ({ page }) => {
    await page.goto(`http://localhost:4173${path}`, { waitUntil: 'networkidle' });
    await expect(page).toHaveScreenshot({
      fullPage: true,
      // Non-deterministic regions are masked, not hoped about:
      // WebGL canvases render differently per GPU/driver; videos advance frames.
      mask: [page.locator('canvas'), page.locator('video')],
    });
  });
}

test('visual page count', () => {
  // Anti-vacuous-gate habit: the log states coverage; zero pages = failure.
  console.log(`visual: screenshotting ${PAGES.length} pages x 2 viewports`);
  expect(PAGES.length).toBeGreaterThan(0);
});
```

Baselines land in `tests/visual.spec.ts-snapshots/` named per test + project + platform (`...-desktop-linux.png`). Commit them; review diffs in PRs like any other change.

## The workflow job

```yaml
  visual:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci
      - run: npm run build            # seed/deterministic data source in CI
      - run: npx playwright install --with-deps chromium
      - if: ${{ inputs.update_snapshots }}
        run: npx playwright test --update-snapshots
      - if: ${{ inputs.update_snapshots }}
        uses: actions/upload-artifact@v4
        with: { name: visual-baselines, path: tests/*-snapshots/ }
      - if: ${{ !inputs.update_snapshots }}
        run: npx playwright test
      - if: ${{ failure() }}
        uses: actions/upload-artifact@v4
        with: { name: visual-diffs, path: test-results/ }
```

Add a `workflow_dispatch` trigger with a boolean `update_snapshots` input to the workflow. The job joins Lighthouse, Biome, and axe in the same ci.yml — perf-ci-gates owns that file; this reference owns what the job runs.

## The baseline-update flow (CI-generated)

Baselines must be rendered by the SAME environment that will compare them. The flow, entirely in CI:

1. Trigger the workflow manually with `update_snapshots: true` (`gh workflow run ci.yml -f update_snapshots=true`).
2. The job renders fresh baselines on the CI runner and uploads them as the `visual-baselines` artifact.
3. Download the artifact (`gh run download`), commit the PNGs under `tests/*-snapshots/`, push.
4. The next normal run compares against them — green means visually unchanged since the commit that blessed the baselines.

Repeat the flow on: a Playwright version bump, an INTENDED redesign, or a rendering-stack change (fonts, tokens). The commit message should say WHY baselines changed — a baseline update with no stated reason is how regressions get blessed.

Local alternative when CI round-trips are too slow: the official `mcr.microsoft.com/playwright` Docker image PINNED to the exact Playwright version renders like the runner (both Ubuntu); never generate baselines on the host OS.

## Flake discipline

- Mask everything non-deterministic: WebGL canvases (GPU/driver variance), videos (frame advance), third-party iframes (Turnstile), live dates.
- Self-hosted fonts only — a CDN font that fails to load in CI shifts every glyph. (This stack already self-hosts brand fonts as woff2.)
- `networkidle` is acceptable here (static site, no sockets); pages with Turnstile need `domcontentloaded` + explicit waits instead — that page earns its mask or its exclusion.
- Keep the page set SMALL and load-bearing (home, the conversion page, the media-heavy page). Every page added is baseline maintenance forever.

## Reading a failure

A red visual job attaches a `visual-diffs` artifact: expected, actual, and diff PNGs per failure. Three outcomes: (a) unintended regression — fix the code; (b) intended change — rerun the update flow, commit baselines WITH the reason; (c) environment flake — tighten masks/thresholds, and if a page flakes twice, redesign its test before muting it. Muting a visual test without a replacement is deleting the gate.

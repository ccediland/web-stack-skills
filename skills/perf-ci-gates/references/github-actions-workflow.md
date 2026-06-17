# GitHub Actions Workflow

> The full ci.yml for the two gates, plus the caching, branch-protection, and report-output wiring, and the runner and storage variants. Targets Astro 6 plus Cloudflare on Node 22.

## Table of contents
- Full ci.yml
- Why these settings
- Runner variant: raw lhci autorun
- Storage variant: self-hosted lhci server
- Branch protection

## Full ci.yml

```yaml
name: ci
on:
  push:
    branches: [main]
  pull_request: {}

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npx @biomejs/biome ci --reporter=github

  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 20
      - uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm
      - run: npm ci
      - run: npm run build
      - uses: treosh/lighthouse-ci-action@v12
        with:
          configPath: ./lighthouserc.json
          uploadArtifacts: true
          temporaryPublicStorage: true
```

## Why these settings

| Setting | Reason |
|---|---|
| two parallel jobs | the quality gate is cheap and fails fast on a typo; the lighthouse gate is heavy. Separation means a lint failure does not burn a Lighthouse run, and each reports its own status |
| node-version 22 | Astro 6 requires Node 22+; matching it in CI avoids a version skew between local and CI builds |
| cache: npm | caches the npm download cache keyed on package-lock.json, cutting install time. Use cache pnpm or yarn if that is the package manager |
| fetch-depth 20 | shallow clones break LHCI git-ancestor detection and cause "Could not find hash" errors; 20 gives enough history for base-branch comparison |
| ref pull_request.head.sha | audits the actual PR head commit rather than the merge commit GitHub creates |
| uploadArtifacts plus temporaryPublicStorage | stores the full report as a downloadable Actions artifact and a public 7-day link, both with no GitHub App |
| concurrency cancel-in-progress | a new push to the same ref cancels the prior run, saving minutes on rapid pushes |

Ubuntu runners ship Chrome at /usr/bin/google-chrome, so no browser install step is needed. Biome does not auto-enable the GitHub reporter inside Actions despite the docs implying it, so `--reporter=github` is passed explicitly or inline annotations do not appear.

## Runner variant: raw lhci autorun

If you prefer the CLI to the action, it reuses the same lighthouserc.json. It needs the LHCI GitHub App installed and either an app token or a personal access token to set PR status.

```yaml
  lighthouse:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}
          fetch-depth: 20
      - uses: actions/setup-node@v4
        with: { node-version: 22, cache: npm }
      - run: npm ci && npm run build
      - run: npm install -g @lhci/cli@0.15.1
      - run: lhci autorun
        env:
          LHCI_GITHUB_APP_TOKEN: ${{ secrets.LHCI_GITHUB_APP_TOKEN }}
```

The token alternatives: install the LHCI GitHub App and use the app token it shows (short-lived, scoped to status checks, safer for public repos), or create a personal access token with the `repo:status` scope and pass it as `LHCI_GITHUB_TOKEN`. The treosh action is the better default for a GitHub-Actions-native setup because it needs neither.

## Storage variant: self-hosted lhci server

`temporary-public-storage` is the zero-infrastructure default; reports are public and auto-deleted after 7 days. For durable history and trend dashboards, self-host `@lhci/server` (a Docker image, suitable for a small self-host such as PikaPods) and point the upload at it.

```json
"upload": {
  "target": "lhci",
  "serverBaseUrl": "https://lhci.example.com",
  "token": "BUILD_TOKEN_FROM_lhci_wizard"
}
```

Run `lhci wizard` once to create a project on the server; it issues a write-only build token (safe to expose as a CI secret, it cannot read or delete history) and a separate admin token (never put the admin token in CI). With the treosh action, pass `serverBaseUrl` and `serverToken` inputs instead of the upload block.

## Branch protection

The gates only block merges when they are required status checks. In repository settings, under branch protection for main, require the `quality` and `lighthouse` checks to pass before merging. Without this, a red check is advisory and a PR can still merge.

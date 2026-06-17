---
name: perf-ci-gates
description: Use when adding Core Web Vitals and code-quality gates to a website CI pipeline — Lighthouse CI with a budget.json for LCP and CLS and TBT and byte-weight thresholds, plus Biome as the single lint and format gate, wired into GitHub Actions. Trigger on requests to set up performance budgets, gate pull requests on Lighthouse scores, add LHCI to CI, or configure Biome. Not for runtime profiling, bundler tuning, or local-only linting.
---

# perf-ci-gates

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

Lighthouse CI (`@lhci/cli`) with a `budget.json`, plus Biome as the single lint and format gate, in GitHub Actions.

## Version pins (seed — re-verify)

- `@lhci/cli@0.15.1` (Lighthouse 12.6.1)
- `@biomejs/biome@2.5.0`
- Node 22.12+

## Outline (author at turn 5)

- `budget.json` with LCP under 2500, CLS under 0.1, TBT under 200, plus byte-weight; `numberOfRuns: 3`; LHCI App PR status; Biome as the single gate; mobile preset.
- references — GitHub Actions workflow, `budget.json`, `biome.json`, serving `dist/` for LHCI.
- Gotchas — LHCI 0.15.x pins LH 12.6.1; `autorun` uses a random port (serve `dist/` explicitly); Biome covers Astro templates partially; treat high-variance metrics as warn; Cloudflare Pages defaults Node 18 (set NODE_VERSION 22).
- Generic CWV gate for any site in CI.

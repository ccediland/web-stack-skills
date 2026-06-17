# Core Web Vitals Thresholds and Lab Budgets

> The official field Core Web Vitals thresholds, why the lab gate cannot use them directly, the lab proxies, and the recommended regression-oriented lab budgets for mobile and desktop. Field thresholds are unchanged as of mid-2026.

## Field thresholds (the truth, but not the gate)

These are Google's official "good" thresholds, measured at the 75th percentile of real-user (field) data. They are unchanged as of mid-2026.

| Metric | Good | Poor | Measurable in lab |
|---|---|---|---|
| Largest Contentful Paint (LCP) | 2.5s or less | over 4.0s | yes |
| Interaction to Next Paint (INP) | 200ms or less | over 500ms | no, field only |
| Cumulative Layout Shift (CLS) | 0.1 or less | over 0.25 | yes |

A page passes a metric only when at least 75 percent of real visits meet the good threshold. INP replaced First Input Delay as a Core Web Vital on 2024-03-12.

## The 2.0s claim is false

A circulating claim says Google tightened the LCP good threshold to 2.0s in 2026 and reclassified INP to a "primary" signal via a March 2026 Search Central post. No Google primary source confirms this; web.dev and Google Search Central still document LCP 2.5s, INP 200ms, CLS 0.1, and Google does not use a primary-versus-supplementary framing for Core Web Vitals. The likely origin is the common practice of setting a personal early-warning alert at 80 percent of a threshold (LCP 2.0s, INP 160ms), which is a self-imposed buffer, not a Google bar. Do not encode 2.0s as a threshold.

## Why the lab gate uses different numbers

Lighthouse runs in a lab: a synthetic device with simulated throttling, no real users. It cannot measure INP, which requires real interaction. So the gate does two things differently from the field:

- It substitutes Total Blocking Time (TBT) for INP. TBT sums the main-thread blocking time (the portion of each task over 50ms) between First Contentful Paint and Time to Interactive. It correlates with INP: reducing TBT generally reduces INP. TBT is lab-only and is not itself a Core Web Vital.
- It uses regression-oriented ceilings, not the field thresholds. The gate's job is to fail a pull request that makes things worse, not to certify field "good." Lab LCP under simulated mobile throttling runs higher than field LCP, so a literal 2500ms lab ceiling is reasonable as a do-not-regress line, while diagnostics stay as warnings.

## Recommended lab budgets

Mobile is the Lighthouse default (Moto-G-class device, Slow-4G, 4x CPU slowdown). Desktop is stricter because the simulated hardware is faster.

| Lab metric | Mobile ceiling | Desktop ceiling | Level |
|---|---|---|---|
| largest-contentful-paint | 2500ms | 2000ms | error |
| total-blocking-time (INP proxy) | 300ms | 200ms | error |
| cumulative-layout-shift | 0.1 | 0.1 | error |
| first-contentful-paint | 2000ms | 1500ms | warn |
| speed-index | 3400ms | 2600ms | warn |
| interactive (TTI) | 3800ms | 2900ms | warn |

Category floors complement the metric budgets: performance error at minScore 0.9, accessibility error at 0.95, best-practices and SEO warn at 0.9 (the seo-aeo-schema skill owns the hard SEO gate).

## Calibration

Do not adopt the table verbatim if the site is already far from it. Measure the current values first (run Lighthouse locally a few times, take the median), set each error-level ceiling to roughly 110 to 115 percent of the current value so the gate passes on day one, then ratchet the ceilings down over successive PRs as the site improves. A gate that fails on the first commit gets disabled; a gate calibrated just above current values catches the next regression and tightens over time.

The visuals skills (WebGL atmosphere, Rive, motion) deliberately add main-thread and asset weight. Expect TBT and image budgets to need a deliberate loosening when those land, decided once rather than fought per commit, with the field INP watched in real-user monitoring as the real check.

## Flake control

Lab metrics vary run to run, more so on shared CI runners. Three levers keep the gate stable:

- Run `numberOfRuns: 5` and let LHCI assert against the median; the median of 5 is roughly twice as stable as a single run.
- Gate as error only the stable, high-headroom metrics; keep diagnostics (FCP, Speed Index, TTI) as warnings.
- Prefer resource-weight budgets (script and image kilobytes) as hard gates where possible: byte weight is nearly deterministic across runs, unlike timing metrics, so it catches real regressions without timing noise.

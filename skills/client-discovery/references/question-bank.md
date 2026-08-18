---
title: The question bank — six domains, every question tied to its consuming skill
summary: The full intake bank for phase 1. A coverage structure for an adaptive interview, never a script. Each question names the catalog skill (or recipe) that consumes the answer — a question nothing consumes was deleted.
last_updated: 2026-08-18
applies_to: web-stack catalog v2 (19 skills) · interview-first intake, exportable form in intake-form.md
---

# The question bank

> Use it as a coverage structure: follow the client's thread, skip what their material already answers, and sweep the bank as a CHECKLIST before closing phase 1. Reading it aloud top-to-bottom is the one wrong way to use it.

## Contents

- How to run it
- Domain 1 — Business
- Domain 2 — Audience
- Domain 3 — Content
- Domain 4 — Functional
- Domain 5 — Brand
- Domain 6 — Measurement
- The STOP rule
- Coverage checklist

## How to run it

- Open with the business domain — it produces the primary-conversion answer that half the other domains hang from.
- Every question below states WHO CONSUMES the answer. When a client answer is vague, push exactly as hard as the consumer needs: "people should find us" is not actionable; "someone whose pet just died searches at 2am and needs to know we answer" feeds three skills.
- Record answers in the client's own words when they state business rules, prohibitions, or claims — downstream truth-gating (brand-canon-ingest voice rules, conversion-patterns trust signals) needs the quotable source.
- A NO or a "we don't have that" is an answer, not a gap: it produces verdicts and deferrals (no brand repo → tokens-only path; no content → placeholder governance).

## Domain 1 — Business

| Question | Consumed by |
|---|---|
| What does the business sell, and what does a SUCCESSFUL site visit end in — a form, a WhatsApp message, a call, a purchase, a booking? | conversion-patterns (primary-conversion tree) · forms recipe |
| How do you close customers today, off the web? (phone, WhatsApp, referrals, walk-ins) | conversion-patterns — the primary conversion mirrors how the business ACTUALLY closes |
| What is the one thing this site must do that the current situation does not? | the brief's headline objective; playbook subset selection |
| Are there claims you make to customers that must appear? (guarantees, certifications, differentiators) — what is the SOURCE of each? | conversion-patterns trust signals (canon-gated slots) · seo-aeo-schema |
| Are there things you must NOT say? (regulated claims, competitor mentions, prohibited vocabulary) | brand voice rules if a canon exists; otherwise recorded verbatim in the brief |
| Who decides on this project, and who edits the site after launch? | cms-self-edit (editor reality) · deferral register (who "the owner" is) |

## Domain 2 — Audience

| Question | Consumed by |
|---|---|
| Who arrives at this site, and in what state? (researching calmly, in urgency, comparing vendors, sent by a referral) | conversion-patterns (landing hierarchy) · motion-system (how much motion the state tolerates) |
| What device and network reality? (mobile-heavy MX traffic, office desktops, in-store tablets) | perf-ci-gates budgets · media-optimization tiers |
| What languages do your customers actually use? Is a second locale a launch need or a someday? | i18n-system (skill in or deferred) |
| Any audience with declared accessibility needs, or a contract/tender that requires formal WCAG? | a11y-deep (floor-only versus formal AA) |
| Where do visitors come FROM? (Google search, Maps, social, ads, word of mouth) | seo-aeo-schema scope · analytics recipe (what to measure first) |

## Domain 3 — Content

| Question | Consumed by |
|---|---|
| What content exists TODAY, and in what form? (old site, docs, photos with rights, nothing) | placeholder governance — what is real versus marked placeholder at launch |
| What content is a LIST of similar things? (products, plans, dishes, projects, testimonials) — how many, how often do they change, and who changes them? | data-layer versus cms-self-edit (rows versus documents — the boundary decision) |
| Will anyone non-technical write or edit pages after launch? Which pages, how often? | cms-self-edit (and its content-modeling reference) |
| Is there long-form content — blog, guides, cases? Planned or aspirational? | cms-self-edit collections · seo-aeo-schema (an empty blog is a deferral, not a scaffold) |
| Do you have image/video material with usage rights, or does it need production? | media-optimization (tiers) · deferral register (asset governance) |
| What legal surfaces does the site need? (privacy notice, terms, sector-specific) | legal recipe (playbook) — template-not-advice, owner/lawyer-gated |

## Domain 4 — Functional

| Question | Consumed by |
|---|---|
| Beyond reading, what can a visitor DO? (submit a form, filter a catalog, book, pay, log in, track an order) | the feasibility pass — each verb maps to a skill, a recipe, a flip, or the portal frontier |
| Does any data on the site live somewhere else already? (a spreadsheet, a POS, a database, a booking system) | data-layer (build-time bias; which tier the freshness need buys) |
| How fresh must changing data be? (daily is fine / minutes matter / real-time) | data-layer tiers · the rebuild-cycle recipe |
| Does anything need to be BEHIND a login? Who are the users and how many? | auth-simple ladder — and the client-portal FRONTIER if it is customer accounts with data |
| Any experiment appetite — A/B tests, variants? | edge-logic (client-side A/B) — usually a later composition pass, record the appetite |
| Redirects from an old site or domain moves? | edge-logic (_redirects ladder) |

## Domain 5 — Brand

| Question | Consumed by |
|---|---|
| Does a governed brand source exist? (a brand repo under the interchange contract, a brand book, scattered logos, nothing) | the ENTRY POINT decision — brand-canon-ingest versus astro-css-tokens standalone versus a scoper-then-builder detour recorded in the brief |
| If no brand exists — is building one in scope, or does the site launch on a minimal palette? | detour to brand-canon-scoper (a BRAND brief, not this one) or a tokens-only verdict |
| What sites do you point at as "like this"? What EXACTLY do you like about each? | structural requirements + the visual handoff note (never verbatim reproduction) |
| How much motion does the brand tolerate? (calm and editorial versus kinetic) | motion-system ladder · whether ONE visual skill is even on the table |
| Is there any brand asset that could anchor a signature moment? (an authored animation file, a mascot, a distinctive pattern) | signature-anim versus webgl-atmosfera versus neither — evidence, not taste |

## Domain 6 — Measurement

| Question | Consumed by |
|---|---|
| What number tells you the site WORKED? (leads/week, calls, bookings, sales) | analytics recipe (event vocabulary) · conversion-patterns (instrumentation-first sequence) |
| Do you run ads today or plan to? Which platform? | analytics recipe — the GA4 tombstone has an Ads exception; know early |
| Any existing analytics whose history matters? | migration note in the brief; usually a deferral |
| Who reads the numbers, and how often? | analytics recipe (dashboard reality — a number nobody reads is not a requirement) |

## The STOP rule

Phase 2 can start once these four are known: the PRIMARY CONVERSION (what a successful visit ends in), the CONTENT INVENTORY (what exists, what is a list, who edits), the BRAND SOURCE (governed repo / material / nothing), and the EDIT EXPECTATION (who touches the site after launch). Everything else can arrive later without blocking the brief — mark it pending, do not hold the pipeline hostage to a complete bank.

## Coverage checklist

Before closing phase 1, sweep: every domain visited or consciously skipped (client material already answered it) · the four STOP-rule answers recorded · every business rule or claim captured in the client's words · every "we don't have that" turned into a note, not silence. An interview that cannot check these is not done, however pleasant it was.

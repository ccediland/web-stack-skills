---
title: Worked example — the Furever fixture brief, run retroactively
summary: The four phases run once, end to end, against real client material (the Furever fixture's experience brief and brand repo). Retroactive by design — the build existed first — which makes every verdict checkable against what actually shipped. Sanitized for a public catalog.
last_updated: 2026-08-18
applies_to: web-stack catalog v2 · fixture evidence 2026-08 (furever-web branch fixture, four CI gates green)
---

# Worked example — Furever, retroactively

> This example is HONEST about its direction: the fixture was built first and the discovery ran backwards over the client's real material (a closed experience brief, a governed brand repo, an existing production site). That inversion is what makes it useful — every verdict below can be checked against a build that already shipped its gates green. It is also SANITIZED: client-sensitive business rules are summarized, never quoted, because this catalog is public and the real brief belongs in the site's own repo. Your briefs will quote; this example points.

## Contents

- What the client material was (phases 1–2)
- The brief, emitted
- Feasibility verdicts (phase 3)
- Deferral register excerpt (phase 4)
- What the retroactive run teaches

## What the client material was (phases 1–2)

- No live interview was needed — the material already covered the bank: a closed EXPERIENCE BRIEF (conversion model, journey, information architecture, page anatomy, SEO targets, copy direction), a GOVERNED BRAND REPO under the interchange contract (tokens, schemes, assets, voice rules with IDs), and the production site's own resident doc. This is the "client hands you a written doc" capture path at its richest.
- The coverage checklist still ran, and still earned its place: the material answered five domains fully but MEASUREMENT only partially (a conversion model existed; no analytics account, no event history) — a real gap that became a deferral, not a guess.
- Format note: the brand repo did not enter the brief as content — it entered as the BRAND SOURCE answer ("governed repo under contract"), which flipped the entry-point verdict to brand-canon-ingest and made every visual question downstream evidence-gated instead of taste-gated.

## The brief, emitted

Compressed to its load-bearing lines (the real artifact follows brief-schema.md in full):

    ## Objective
    A site that makes a grieving pet owner feel accompanied and guided until they
    reach out — premium warmth, never clinical, never festive.

    ## Primary conversion
    WhatsApp message. Secondary — a phone call. The whole site funnels to one act,
    mirroring how the business actually closes. (Client's own conversion model,
    stated in their experience brief.)

    ## Audience
    B2C pet owners in an emotionally acute state, mobile-heavy MX traffic. Spanish
    at launch; a second locale is a someday, not a launch need. No formal WCAG
    mandate — the catalog's own floor applies.

    ## Content inventory
    A LIST — a keepsakes/plans catalog (rows, client-updated) — plus long-form
    pillar guides and a blog (documents, client-edited after launch), standard
    pages, and the legal surfaces. Image bank pending rights governance.

    ## Functional requirements
    Contact capture with spam protection · WhatsApp and call taps, measured ·
    catalog rendered from business data · client self-edit of blog/guides ·
    order tracking by folio (customer-facing status app) · private memorial
    pages per customer · experiment appetite (A/B on the closing ask).

    ## Brand source
    Governed brand repo under the interchange contract — entry point is
    brand-canon-ingest, step 1 of the canonical order. Voice carries hard
    prohibition rules with IDs (summarized here — e.g. no service-time promises,
    no absolute guarantees, no anglicisms in es-MX copy); the brief records them
    by reference to the canon, which is their single source of truth.

    ## Measurement
    The number — WhatsApp conversations started per week. No analytics account
    exists yet (deferral D-3); the event vocabulary is specified so wiring is a
    flip, not a redesign.

## Feasibility verdicts (phase 3)

The real table, against catalog v2 — every verdict checkable against the fixture:

| Item | Verdict | Owner | Flip or condition | Cost accepted |
|---|---|---|---|---|
| Brand-true styling from the governed repo | feasible-as-is | brand-canon-ingest → astro-css-tokens | — | ingest re-run on canon changes |
| Contact capture + spam protection | feasible-as-is | forms recipe (playbook) | — | — |
| WhatsApp/call taps, measured | feasible-as-is | conversion-patterns + analytics recipe | — | events dark until the account exists (D-3) |
| Catalog from business data | feasible-as-is | data-layer tier 1 | — | freshness = rebuild cycle (webhook, ~minutes) |
| "Prices reflect instantly" (if ever asked) | feasible-with-flip | data-layer tier 3 | live collections force on-demand | deploy shape changes — Worker gains main |
| Client edits blog/guides | feasible-as-is | cms-self-edit | — | drafts-branch publishing, not instant |
| Atmospheric hero | feasible-as-is | webgl-atmosfera | canon carries the atmosphere rule; no authored .riv exists | the software-GL guard is mandatory |
| A/B on the closing ask | feasible-as-is | edge-logic (client-side) | — | needs analytics live to read results |
| Order tracking by folio | out-of-scope | client-portal frontier (playbook) | — | separate build decision — auth + per-customer data change the deploy shape |
| Private memorial pages | out-of-scope | client-portal frontier (playbook) | — | same frontier, same reason |
| Aviso de privacidad | feasible-as-is | legal recipe (playbook) | — | template-not-advice; lawyer/owner gate before real |

The two out-of-scope rows are the frontier doing its job: both are real client needs, neither is a SITE feature, and stamping them out-of-scope WITH an address (the portal frontier note in the playbook) is what keeps them decisions instead of scope creep.

## Deferral register excerpt (phase 4)

Three entries from the fixture's real governance, in the register schema:

    ### D-1 — image bank
    - type — functional
    - what was cut — real photography; the build ships marked placeholders
    - decided by — owner, 2026-08
    - context — usage-rights governance pending; fabricating imagery is prohibited
      by the canon's truth rules
    - violates or blocks — media-optimization runs on synthetic sets; conversion
      trust signals that need real proof stay empty slots
    - revisit when — the owner's asset governance closes

    ### D-2 — aviso de privacidad (real content)
    - type — functional
    - what was cut — the legal page ships as the template with visible slot markers
    - decided by — owner, 2026-08
    - context — template-not-advice; the owner and a lawyer fill the slots
    - violates or blocks — the legal recipe's owner-gate is open; the consent
      checkbox links to a marked template
    - revisit when — owner/lawyer review happens

    ### D-3 — analytics account
    - type — functional
    - what was cut — live event delivery (the wiring ships env-gated and dark)
    - decided by — owner, 2026-08
    - context — no Umami account yet; wiring shipped honest-stub so flipping it
      on is configuration, not construction
    - violates or blocks — conversion-patterns cannot run its instrumentation-first
      loop until this closes; the A/B has no readout
    - revisit when — the account exists and the site id lands in the env

Note what the type field buys: all three are FUNCTIONAL, each names the discipline its absence touches, and D-3's entry is what makes "the A/B has no readout" a documented known hole instead of a surprise at the first optimization pass.

## What the retroactive run teaches

- Rich client material can REPLACE the interview but not the checklist — the measurement gap only surfaced because the coverage sweep ran anyway.
- The brand-source question is the highest-leverage question in the bank: one answer flipped the entry point, the visual decision, and the voice governance in a single stroke.
- Out-of-scope verdicts with an address are client-relationship gold: the portal items stayed in the conversation as futures, not as failures.
- A brief written AFTER a build is a validation instrument; a brief written before one is a steering instrument. Same schema, same verdicts — which is the point.

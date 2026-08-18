---
title: The brief schema and the deferral register — both artifacts, section by section
summary: The structured site brief (sections, the per-item verdict format, emission rules) and the deferral register schema the site repo keeps. The brief is the skill's product; the register is the site's property.
last_updated: 2026-08-18
applies_to: web-stack catalog v2 · verdicts issued against stack-integration-playbook (canonical order, stack map, recipes)
---

# The brief schema and the deferral register

> Two artifacts leave this skill: the BRIEF (one markdown file, the build's requirements source) and the DEFERRAL REGISTER (a schema the site repo instantiates and keeps). The brief without verdicts is notes; the register without owner context is a wishlist. Neither counts as done in those states.

## Contents

- The brief — section by section
- The per-item verdict format
- The deferral register schema
- Emission rules
- What the brief is not

## The brief — section by section

One file, `site-brief.md`, living in the site repo (or the project folder that will become it). Sections in order; every section mandatory, one-line answers fine, "none" is an answer:

    # Site brief — [client]
    date / interviewed / materials received

    ## Objective
    [The one thing this site must do, in one sentence. The headline the whole build serves.]

    ## Primary conversion
    [What a successful visit ends in, mirrored from how the business actually closes.
     Secondary action if one exists. Source quotes from the client.]

    ## Audience
    [Who arrives, in what state, on what device/network reality. Languages. A11y needs.]

    ## Content inventory
    [What exists and its form · what is a LIST (rows) versus PAGES (documents) ·
     who edits what after launch · legal surfaces · asset/rights reality.]

    ## Functional requirements
    [Each visitor verb as its own item — these are the rows the verdict table stamps.]

    ## Brand source
    [Governed repo under contract / material without governance / nothing.
     The entry-point decision this forces. Visual-intent handoff notes attached here.]

    ## Measurement
    [The number that says it worked · event appetite · ads reality.]

    ## Feasibility verdicts
    [The table below — every functional requirement and every non-obvious content/brand
     item gets a row.]

    ## Deferrals
    [Pointer to the register (or the register itself while the repo is young).]

    ## Explicitly out
    [What was discussed and is NOT in scope, so its absence reads as a decision.]

## The per-item verdict format

One row per item; the four verdicts are closed vocabulary:

| Item | Verdict | Owner (skill/recipe) | Flip or condition | Cost the owner accepts |
|---|---|---|---|---|
| Contact form with spam protection | feasible-as-is | forms recipe (playbook) | — | — |
| Catalog from their spreadsheet | feasible-as-is | data-layer tier 1 | — | rebuild-on-change freshness |
| "Update prices instantly" | feasible-with-flip | data-layer tier 3 | live collections force on-demand | deploy shape changes; Worker gains main |
| Customer login to see orders | out-of-scope | client-portal frontier (playbook) | — | separate build decision, not a site feature |
| Blog "when we have time" | deferred | cms-self-edit | reopen when 3 posts exist as drafts | empty scaffold shipped = noindex placeholder |

- feasible-as-is and feasible-with-flip MUST name the owning skill or recipe — an unowned yes is a guess.
- feasible-with-flip MUST state the flip's cost in the owner's terms (money, deploy shape, maintenance); the verdict exists so the owner accepts the cost consciously.
- out-of-scope points at where the need would live (another build, a documented frontier, a different tool) — never just "no".
- The verdict facts come from the playbook (canonical order, stack-integration-map, recipes) and the owning skills' documented flips. When the fact is missing there, the verdict is "pending — playbook gap reported", not an improvised answer.

## The deferral register schema

Lives IN THE SITE REPO (a `## Deferrals` section of the brief while young; its own `deferrals.md` when it outgrows that). Same pattern as the brand contract's projections file — the catalog supplies the schema, the consuming project holds the state. One entry per deferral:

    ### [item, short name]
    - type — aesthetic | functional
    - what was cut — [one line]
    - decided by — [owner name], [date]
    - context — [the owner's reasoning, quoted or paraphrased close]
    - violates or blocks — [which skill's discipline the absence touches, if functional; "none" if aesthetic]
    - revisit when — [a date, a metric threshold, or a governance event — never "someday"]

- The type field is load-bearing: functional deferrals name the skill whose discipline the hole touches (no analytics → conversion-patterns cannot run its loop; no legal page → the legal recipe's owner-gate is pending), so later composition passes read them as known holes.
- Revisit conditions must be checkable: "when we have budget" is an owner's honest answer and acceptable IF dated for review; "someday" is not an entry.

## Emission rules

- Emit in the client's language for client-facing sections (objective, conversion, deferral context) and keep skill/recipe names in English — the mixed register is deliberate and matches how the catalog documents itself.
- Quote the client verbatim wherever a business rule, prohibition, or claim enters the brief — downstream truth-gating needs the source, and paraphrase is where prohibitions get lost.
- Date the volatile: audience numbers, ad plans, content counts. The brief's structure is durable; its market facts are not.
- The brief is REGENERABLE from a better interview; the register is NOT regenerable (it holds decisions). Losing the brief costs a re-interview; losing the register costs governance — which is why it lives in the repo, versioned.

## What the brief is not

- Not a proposal, a quote, or a contract — it feeds them.
- Not a design: visual intent travels as handoff notes to the visual stage, never as layout decisions made here.
- Not the composition: the playbook's first pass reads the brief and picks the subset; the brief only makes that pass honest.

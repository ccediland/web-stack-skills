---
title: Voice and copy discipline
summary: How to load the brand's resident set (canon.json plus both keystones) for every copy or design-judgment task, truth-gate claims, and cite canon rules by ID instead of restating them.
last_updated: 2026-08-17
applies_to: "brand-system-skills 0.6.0"
---

# Voice and copy discipline

Load when writing or reviewing ANY site copy — headlines, body, microcopy, error states, meta descriptions — or when making a design judgment the canon governs.

## The resident set loads together

The brand repo ships a resident set for AI work: `canon/canon.json` (the machine mirror — every rule ID, scheme declaration, and structural fact in queryable form) plus TWO keystones at the repo root — `<brand>-keystone.md` (the verbal brain: how the brand thinks and speaks) and `<brand>-visual-keystone.md` (the design brain: how the brand looks and composes), with `satellites/asset-index.md` as the consultation map.

Load canon.json AND both keystones together, every time. The failure mode of partial loading is real: verbal-only sessions drift on design-adjacent copy (captions, alt text, CTA styling words), visual-only sessions produce on-palette pages that speak off-brand. The set is designed as one brain in three files.

## Truth-gating — copy claims trace to a source

Every factual claim in site copy must trace to the canon or to the data sources the brand's `satellites/data-map.md` names. Concretely:

- Prices, plan names, coverage, hours, phone numbers: read from the data source at build or runtime — never typed into a component. Plan NAMES can be canon (check the canon before renaming anything).
- Brand claims (what the brand promises, refuses to promise, how it describes its work): the keystone's voice sections and the canon's essence layer decide. The anti-promise list is as binding as the promise list.
- If the copy needs a fact no source holds, that is a GAP for the brand owner — log it, don't invent it. Fabricated specifics (response times, guarantees, counts) are the highest-risk drift.

## Cite rules by ID — never restate them

Canon rules carry stable IDs (`G-*` for grammar rules, `ALGO-*` for derivation algorithms). When a site decision leans on one, cite the ID — in code comments, PR descriptions, the site's decision log:

```astro
{/* dark header: single luminous ink per G-SCHEME-02 — do not add a second ink */}
```

Never copy a rule's text into the site repo. Restated rules fork the truth: the canon edition evolves under the brand's gates, the pasted edition rots silently. The site repo holds pointers (IDs + the brand repo path), the brand repo holds rules.

## Lexicon and register

The keystone carries the brand's lexicon (words it uses, words it never uses) and register per surface. Two operational habits:

- Run new copy against the lexicon's never-list before committing — a single off-lexicon word in a headline is a canon violation, not a style choice.
- Where the brand defines register variants (e.g. a service register and a memorial register mapped to scheme B), the copy register and the visual scheme switch TOGETHER — a memorial-register section in service-scheme colors is a mixed signal the canon forbids.

## When the canon is silent

A copy or design question the canon doesn't answer is not a license to improvise permanently. Decide minimally, mark the decision in the site's decision log as canon-silent, and surface it to the brand owner — the brand repo's promotion path can then ratify it into the canon (or reject it). See `governance-projections.md` for the promotion path.

## Limitations

- This document does NOT restate any brand's actual voice rules — they live only in that brand's keystones and canon.
- Legal copy (privacy notices, terms) follows law first; the voice discipline styles it but never overrides required content.

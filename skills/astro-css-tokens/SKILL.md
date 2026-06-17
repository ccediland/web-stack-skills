---
name: astro-css-tokens
description: Use when wiring a design-token pipeline into an Astro site styled with Tailwind v4 — a single tokens.json in DTCG format compiled by Style Dictionary into two consumers, flat CSS variables plus a Tailwind @theme block, so tokens survive a framework swap with no lock-in. Trigger on requests to set up design tokens, theme an Astro and Tailwind v4 project, generate CSS variables from a token source, or establish base and semantic and component token layers. Not for component-level styling decisions, palette design, or non-Astro stacks.
---

# astro-css-tokens

> Status — skeleton (Phase 0). Full content is authored at this skill's turn 5 of the 5-turn cadence.

## Verdict (seed, 2026-06-16 — re-verify at research)

Tailwind v4 `@theme` plus Style Dictionary compiling `tokens.json` (DTCG) into two consumers — flat CSS variables and a Tailwind `@theme` block. The CSS variables outlive a Tailwind swap, so there is no lock-in.

## Version pins (seed — re-verify)

- `tailwindcss@4.3.1`, `@tailwindcss/vite@4.3.1`
- `style-dictionary` v4

## Outline (author at turn 5)

- The two-consumer token bridge (CSS vars + `@theme`) and three token layers (base, semantic, component) via DTCG references.
- references — Style Dictionary config, a DTCG `tokens.json` example, `global.css` wiring, OKLCH color notes, gotchas.
- Gotchas — never `@astrojs/tailwind` with v4 (use `@tailwindcss/vite`); avoid `@apply` (if unavoidable in scoped styles, `@reference "tailwindcss"`); `@theme` replaces `tailwind.config.js`; pin versions.
- Generic example only — no project-specific tokens.

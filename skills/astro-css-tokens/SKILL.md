---
name: astro-css-tokens
description: Use when wiring a design-token pipeline into an Astro site styled with Tailwind v4 — a single tokens.json in DTCG format compiled by Style Dictionary into two consumers, flat CSS variables plus a Tailwind @theme block, so tokens survive a framework swap with no lock-in. Trigger on requests to set up design tokens, theme an Astro and Tailwind v4 project, generate CSS variables from a token source, or establish base and semantic and component token layers. Not for component-level styling decisions, palette design, or non-Astro stacks.
---

# astro-css-tokens

## Verdict

Use when you have two or more CSS consumers: Tailwind utilities plus hand-authored CSS, or two separate apps reading the same tokens. If Tailwind is the only consumer, `@theme` hand-authored is enough — Style Dictionary is unnecessary overhead.

Architecture:

```
tokens/
  base.json       raw values (OKLCH, rem, px)
  semantic.json   intent aliases  {base.color.blue.500}
  component.json  component aliases  {semantic.color.primary}
      |
      v
  Style Dictionary v5  (node build.mjs, prebuild npm script)
      |
      ├─ src/styles/tokens.css   :root { --ds-*: oklch(...) / var(--ds-*) }
      └─ src/styles/theme.css    @theme inline { --color-*: var(--ds-semantic-color-*); ... }
                                        |
                              Tailwind reads theme.css
                              Hand CSS + 2nd consumer read tokens.css
```

One `[data-theme="dark"]` block in `tokens.css` re-themes every consumer at once.

## Prerequisite: two or more consumers

This skill is justified only when at least two independent systems read the compiled tokens. Single-consumer projects should author `@theme` variables by hand.

## Pinned versions

| Package | Pin | Note |
|---|---|---|
| `style-dictionary` | 5.4.4 | v5 only — config format changed from v4 |
| `tailwindcss` | 4.3.1 | — |
| `@tailwindcss/postcss` | 4.3.0 | NOT `@tailwindcss/vite` — see wiring ref |
| `astro` | 6.4.7 | — |
| Node | ≥ 22.12 | SD v5 requirement |

## Three token layers

| Layer | File | Role |
|---|---|---|
| base | `tokens/base.json` | raw values — OKLCH color, rem spacing, px |
| semantic | `tokens/semantic.json` | intent aliases — `{base.color.blue.500}` |
| component | `tokens/component.json` | component aliases — `{semantic.color.primary}` |

Only semantic and component layers map to Tailwind utilities in `theme.css`. Base values are implementation detail.

## Setup — in order

### 1. Install

```bash
npm install -D style-dictionary@5.4.4 tailwindcss@4.3.1 @tailwindcss/postcss@4.3.0
```

### 2. Author tokens

Create the three files under `tokens/`. See `references/tokens-example.md` for the verified minimal set.

### 3. Configure Style Dictionary

Create `build.mjs` at the project root. See `references/style-dictionary-config.md` for the full verified config including the custom `css/tailwind-theme` format.

Add npm hooks so SD runs before every dev and build:

```json
"scripts": {
  "prebuild": "node build.mjs",
  "predev":   "node build.mjs"
}
```

### 4. Wire Astro

Create `postcss.config.mjs` and import both CSS files in the global stylesheet. See `references/astro-tailwind-wiring.md` for exact paths, import order, and the no-flash dark-mode inline script.

### 5. Dark mode seam

Add a `@custom-variant dark` declaration and a `[data-theme="dark"]` block that overrides only the `--ds-*` base values. Full multi-theme builds are out of scope for this skill.

## Gotchas

- Never use the SD transform `color/css`. It converts OKLCH to hex via gamut mapping. Use name-only transforms: `attribute/cti` and `name/kebab` only.
- Never suffix an alias with `.value` inside `$value`. SD v5 rejects it.
- Alias refs must point to leaf tokens, not to a group node.
- `$value` must be a plain string, never an object. SD splits object values into `-value`/`-unit` sub-vars (bug #1398/#1494).
- Import `tokens.css` and `theme.css` exactly once in the entry stylesheet. Duplicate imports produce duplicate var declarations.
- `@tailwindcss/vite` breaks on Astro 6 rolldown-vite (issue #16542, open as of Jun 2026). See `references/astro-tailwind-wiring.md` for the switch-back trigger.

## Limitations

- Dark mode is a seam only: one `[data-theme="dark"]` override block. Full multi-theme SD output is a separate concern.
- `outputReferences: true` resolves aliases only within the same output file. Cross-file aliases land as static values in that file.

## References (load when relevant)

- `references/style-dictionary-config.md` — verified `build.mjs`, SD v5 config, custom format, package.json hooks, gotchas
- `references/tokens-example.md` — verified 3-layer `tokens.json`, DTCG authoring rules, compiled output examples
- `references/astro-tailwind-wiring.md` — `postcss.config.mjs`, global CSS import order, no-flash dark script, scoped `@apply`, switch-back trigger
- `references/color-oklch.md` — OKLCH rationale, no-fallback policy, browser support baseline, SD preservation

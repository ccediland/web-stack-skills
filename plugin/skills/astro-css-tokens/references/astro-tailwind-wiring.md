# Astro + Tailwind v4 wiring

Load when configuring the Tailwind Vite plugin, global CSS, dark mode, or scoped @apply.

## @tailwindcss/vite on Astro 7 (review-gate #16542 closed)

On Astro 7 (Vite 8), `@tailwindcss/vite` is the official path — it is what `astro add tailwind` installs. Wire it in `astro.config.mjs`:

```javascript
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  vite: {
    plugins: [tailwindcss()],
  },
});
```

Pin `@tailwindcss/vite` to the same minor as `tailwindcss`. No PostCSS config is needed.

## History and the Astro 6 fallback (@tailwindcss/postcss)

The old "vite plugin broken on Astro 6" verdict was misattributed. Astro 6 never used rolldown-vite — it runs standard Vite 7. The real failure was npm hoisting Vite 8 (rolldown-powered) to satisfy the plugin's wide peer range, crashing it on startup with `Missing field tsconfigPaths`; the underlying plugin bug was [tailwindlabs/tailwindcss#19802](https://github.com/tailwindlabs/tailwindcss/issues/19802) (`aliasOnly: true`, fixed around 4.2.4). pnpm and yarn users were never affected. [withastro/astro#16542](https://github.com/withastro/astro/issues/16542) is closed and the incompatibility is gone in current versions.

If a project must stay on Astro 6, either block the hoist with `"overrides": { "vite": "^7" }` in `package.json` and keep the Vite plugin, or run `@tailwindcss/postcss` through Astro's native PostCSS pipeline:

```javascript
// postcss.config.mjs — Astro 6 fallback only, project root
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

Node floor note: Node 20 support ended at Astro 6.1.0 (not at Astro 7); the effective floor is Node ≥ 22.12.

## Global stylesheet (src/styles/global.css)

The file must START with the Tailwind v4 entry `@import 'tailwindcss'` — that is what makes it a Tailwind root. The v3 directives (`@tailwind base` and friends) no longer exist in v4; with them, the build still SUCCEEDS but Tailwind never runs, no utility is generated, and the literal `@tailwind` line ships to the browser (verified on a real Astro 7 build, 2026-08-17 — the failure is silent). Tailwind collects `@theme` blocks from the whole import graph, so `theme.css` can come after the entry.

```css
/* 1. Tailwind v4 entry — makes this file a Tailwind root */
@import 'tailwindcss';

/* 2. Design tokens — source of truth */
@import './tokens.css';

/* 3. Tailwind @theme inline bridge */
@import './theme.css';

/* 4. Dark mode variant */
@custom-variant dark (&:where([data-theme="dark"], [data-theme="dark"] *));

/* 5. Dark mode token overrides */
[data-theme="dark"] {
  --ds-base-color-neutral-0:   oklch(0.145 0 0);
  --ds-base-color-neutral-900: oklch(1 0 0);
  /* override only the base values that change; aliases resolve automatically */
}
```

Import `global.css` once from your Astro layout:

```astro
---
import '../styles/global.css';
---
```

Never import it from individual pages or components — duplicate imports produce duplicate var declarations.

## No-flash dark mode script

Add this inline script to your layout `<head>` before any CSS loads. It reads the stored preference and sets `data-theme` on `<html>` before first paint:

```astro
<script is:inline>
  (function () {
    const stored = localStorage.getItem('theme');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    const theme = stored ?? (prefersDark ? 'dark' : 'light');
    document.documentElement.setAttribute('data-theme', theme);
  })();
</script>
```

`is:inline` prevents Astro from bundling or deferring the script. It runs synchronously, blocking paint until `data-theme` is set.

## @apply in scoped component styles

Astro scoped `<style>` blocks process in isolation and cannot see the Tailwind utilities defined in your global CSS by default. Use `@reference` to pull them in without emitting duplicate output:

```astro
<style>
  @reference "tailwindcss";

  .btn {
    @apply bg-primary text-white rounded-md px-4 py-2;
  }
</style>
```

`@reference` makes utility definitions available to `@apply` without inserting their CSS into this scope's output.

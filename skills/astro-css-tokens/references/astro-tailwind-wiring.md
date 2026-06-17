# Astro + Tailwind v4 wiring

Load when configuring PostCSS, global CSS, dark mode, or scoped @apply.

## Why @tailwindcss/postcss (not @tailwindcss/vite)

Astro 6 migrated its internal Vite fork to rolldown-vite. The `@tailwindcss/vite` plugin breaks on startup with:

```
Error: Missing field tsconfigPaths
```

Root cause: [withastro/astro#16542](https://github.com/withastro/astro/issues/16542) (open as of Jun 2026, upstream at vite#19802, `aliasOnly: true`).

The workaround is `@tailwindcss/postcss`, which Astro runs through its native PostCSS pipeline without Vite involvement.

Switch-back trigger: close #16542 is confirmed, you can reproduce a clean `astro dev` with `@tailwindcss/vite` pinned to the same minor as `tailwindcss`, and `@tailwindcss/postcss` is removed.

## postcss.config.mjs

Place at the project root (same level as `astro.config.mjs`):

```javascript
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
};
```

No other PostCSS plugins are needed for the token pipeline.

## Global stylesheet (src/styles/global.css)

Import order matters. Tailwind must see `theme.css` before processing utilities.

```css
/* 1. Design tokens — source of truth */
@import './tokens.css';

/* 2. Tailwind @theme inline bridge */
@import './theme.css';

/* 3. Tailwind base styles */
@tailwind base;

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

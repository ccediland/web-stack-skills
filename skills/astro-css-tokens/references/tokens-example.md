# Tokens example — verified 3-layer DTCG

Load when authoring tokens or debugging SD output.

## Minimal token set

Three files under `tokens/`. Each `$value` is a plain string — never an object.

### tokens/base.json

```json
{
  "base": {
    "color": {
      "blue":    { "500": { "$type": "color",     "$value": "oklch(0.546 0.245 264)" } },
      "neutral": {
        "0":   { "$type": "color", "$value": "oklch(1 0 0)" },
        "900": { "$type": "color", "$value": "oklch(0.145 0 0)" }
      },
      "red":     { "500": { "$type": "color",     "$value": "oklch(0.577 0.245 27.3)" } }
    },
    "spacing": {
      "1":  { "$type": "dimension", "$value": "0.25rem" },
      "2":  { "$type": "dimension", "$value": "0.5rem" },
      "4":  { "$type": "dimension", "$value": "1rem" },
      "8":  { "$type": "dimension", "$value": "2rem" }
    },
    "font": {
      "family": {
        "sans": { "$type": "fontFamily", "$value": "Inter, ui-sans-serif, system-ui, sans-serif" }
      },
      "size": {
        "sm":   { "$type": "dimension", "$value": "0.875rem" },
        "base": { "$type": "dimension", "$value": "1rem" },
        "lg":   { "$type": "dimension", "$value": "1.125rem" }
      }
    },
    "radius": {
      "sm":   { "$type": "dimension", "$value": "0.25rem" },
      "md":   { "$type": "dimension", "$value": "0.5rem" },
      "lg":   { "$type": "dimension", "$value": "0.75rem" },
      "full": { "$type": "dimension", "$value": "9999px" }
    }
  }
}
```

### tokens/semantic.json

```json
{
  "semantic": {
    "color": {
      "primary":    { "$type": "color", "$value": "{base.color.blue.500}" },
      "danger":     { "$type": "color", "$value": "{base.color.red.500}" },
      "surface":    { "$type": "color", "$value": "{base.color.neutral.0}" },
      "on-surface": { "$type": "color", "$value": "{base.color.neutral.900}" }
    }
  }
}
```

### tokens/component.json

```json
{
  "component": {
    "color": {
      "button-bg":   { "$type": "color", "$value": "{semantic.color.primary}" },
      "button-text": { "$type": "color", "$value": "{semantic.color.surface}" }
    }
  }
}
```

## DTCG authoring rules

| Rule | Detail |
|---|---|
| `$type` | Required on every leaf token. Use: `color`, `dimension`, `fontFamily`, `fontWeight`, `shadow`. |
| `$value` | Must be a plain string. Never an object — SD splits objects into `-value`/`-unit` sub-vars. |
| Aliases | Written as `{layer.category.name}`. No `.value` suffix. Must point to a leaf token, not a group. |
| Layer prefix | `base`, `semantic`, `component` — must be `path[0]`. SD and `build.mjs` depend on this position. |
| OKLCH strings | Literal strings: `"oklch(0.546 0.245 264)"`. SD preserves them unchanged with name-only transforms. |

## Compiled output — tokens.css (excerpt)

```css
:root {
  --ds-base-color-blue-500: oklch(0.546 0.245 264);
  --ds-base-color-neutral-0: oklch(1 0 0);
  --ds-base-color-neutral-900: oklch(0.145 0 0);
  --ds-base-color-red-500: oklch(0.577 0.245 27.3);
  --ds-semantic-color-primary: var(--ds-base-color-blue-500);
  --ds-semantic-color-danger: var(--ds-base-color-red-500);
  --ds-semantic-color-surface: var(--ds-base-color-neutral-0);
  --ds-semantic-color-on-surface: var(--ds-base-color-neutral-900);
  --ds-component-color-button-bg: var(--ds-semantic-color-primary);
  --ds-component-color-button-text: var(--ds-semantic-color-surface);
  /* spacing, font, radius follow */
}
```

## Compiled output — theme.css (excerpt)

```css
@theme inline {
  --color-primary: var(--ds-semantic-color-primary);
  --color-danger: var(--ds-semantic-color-danger);
  --color-surface: var(--ds-semantic-color-surface);
  --color-on-surface: var(--ds-semantic-color-on-surface);
  --color-button-bg: var(--ds-component-color-button-bg);
  --color-button-text: var(--ds-component-color-button-text);
}
```

Tailwind generates utilities like `bg-primary`, `text-danger`, `text-on-surface` from these. The utility value is `var(--ds-semantic-color-primary)` — runtime-resolved, theme-switchable.

## Composite $value gotcha (bug #1398/#1494)

Object-valued `$value` fields (e.g., `{ "value": "0.25", "unit": "rem" }`) cause SD to emit split vars:

```css
--ds-spacing-1-value: 0.25;
--ds-spacing-1-unit: rem;
```

Always use a plain string: `"$value": "0.25rem"`.

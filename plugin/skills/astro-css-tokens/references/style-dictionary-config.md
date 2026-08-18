# Style Dictionary v5 config

Load when configuring `build.mjs` or debugging SD output.

## Verified build.mjs

```javascript
import StyleDictionary from 'style-dictionary';

// Maps token category (path[1]) to Tailwind v4 CSS namespace
const TW_NAMESPACE = {
  color:           '--color-',
  spacing:         '--spacing-',
  'font-size':     '--text-',
  'font-family':   '--font-',
  'font-weight':   '--font-weight-',
  'line-height':   '--leading-',
  'border-radius': '--radius-',
  shadow:          '--shadow-',
};

// Only semantic and component layers map to Tailwind utilities
const UTILITY_LAYERS = new Set(['semantic', 'component']);

// Custom format: emits @theme inline { --tw-var: var(--ds-*); }
StyleDictionary.registerFormat({
  name: 'css/tailwind-theme',
  format: ({ dictionary }) => {
    const lines = ['@theme inline {'];
    for (const token of dictionary.allTokens) {
      const layer  = token.path[0];          // 'base' | 'semantic' | 'component'
      if (!UTILITY_LAYERS.has(layer)) continue;
      const cat    = token.path[1];          // 'color' | 'spacing' | ...
      const prefix = TW_NAMESPACE[cat];
      if (!prefix) continue;
      // path[2..] gives the token name without layer and category prefixes
      const twVar  = prefix + token.path.slice(2).join('-');
      const dsVar  = '--ds-' + token.path.join('-');
      lines.push(`  ${twVar}: var(${dsVar});`);
    }
    lines.push('}');
    return lines.join('\n') + '\n';
  },
});

const sd = new StyleDictionary({
  source:   ['tokens/**/*.json'],
  usesDtcg: true,
  platforms: {
    css: {
      transforms: ['attribute/cti', 'name/kebab'],
      prefix:     'ds',
      buildPath:  'src/styles/',
      files: [
        {
          // All tokens → :root { --ds-*: oklch(...) / var(--ds-*) }
          destination: 'tokens.css',
          format:      'css/variables',
          options: {
            outputReferences: true,
            usesDtcg:         true,
          },
        },
        {
          // Semantic + component → @theme inline { --color-*: var(--ds-*) }
          destination: 'theme.css',
          format:      'css/tailwind-theme',
        },
      ],
    },
  },
});

await sd.buildAllPlatforms();
```

## What each part does

| Part | Role |
|---|---|
| `TW_NAMESPACE` | maps DTCG category (`path[1]`) to Tailwind v4 CSS variable prefix |
| `UTILITY_LAYERS` | filters out base — utilities only on semantic and component tokens |
| `css/tailwind-theme` | custom format that writes `@theme inline` bridge; reads from compiled `--ds-*` vars |
| `css/variables` | built-in SD format; writes `:root { --ds-*: ... }` |
| `outputReferences: true` | preserves alias chains as `var(--ds-*)` instead of resolving to static values |
| `usesDtcg: true` | enables DTCG `$type`/`$value` parsing |
| `transforms: ['attribute/cti', 'name/kebab']` | assigns path-based attributes + kebab-case variable names; no color transforms |

## package.json hooks

```json
{
  "scripts": {
    "prebuild": "node build.mjs",
    "predev":   "node build.mjs",
    "tokens":   "node build.mjs"
  }
}
```

`prebuild` and `predev` run SD automatically before every `astro build` / `astro dev`. The standalone `tokens` script lets you rebuild outside of those lifecycles.

## Extending TW_NAMESPACE

Add any DTCG category you need. The key must match `token.path[1]` exactly as it appears in your `tokens.json`. The value must be a valid Tailwind v4 CSS variable prefix (see [Tailwind v4 CSS variables reference](https://tailwindcss.com/docs/v4-upgrade#css-variables)).

## Gotchas

- Do not add `color/css` to the transforms array. It converts OKLCH to hex (gamut-mapped, lossy) — see `references/color-oklch.md`.
- Do not use `transformGroup: 'css'`. That group includes `color/css`. Build transforms explicitly.
- SD v5 no longer supports `StyleDictionary.extend()`. Pass the config directly to `new StyleDictionary(config)`.
- `buildPath` must end with `/`. Missing trailing slash silently emits files to the project root.
- `outputReferences: true` only resolves aliases within the same output file. A base token aliased from a file that is not in the output will land as its static value.
- SD v5 requires Node ≥ 22.12. `"type": "module"` in `package.json` is required for the `import` syntax.

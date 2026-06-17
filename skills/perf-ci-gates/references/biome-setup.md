# Biome Setup for an Astro Project

> biome.json for an Astro 6 plus TypeScript project, the Astro template-formatting split with prettier-plugin-astro, the experimental-flag decision, and the biome ci wiring. Biome is the single lint and format tool for JS, TS, CSS, and JSON; prettier-plugin-astro formats .astro templates only.

## Table of contents
- The Astro problem
- biome.json
- prettier for templates
- biome ci behavior
- GitHub reporter
- package.json scripts
- The experimental-flag decision
- Stack-canon exception

## The Astro problem

Biome's full support for HTML-ish languages (Astro, Svelte, Vue) is gated behind `html.experimentalFullSupportEnabled`. With the flag off (the recommended default), Biome extracts only the JS and TS from a `.astro` file (the frontmatter and script) and ignores the template and styles. With the flag on, Biome parses and formats the template too, but that path is experimental: it ships breaking formatting changes between versions, does not yet match prettier, and cross-embedded-language lint rules are still unsupported. For a skill that codifies a stack, the deterministic choice is flag off.

The note: the configuration key is `html.experimentalFullSupportEnabled`. Some release-blog prose writes it as `experimentalFullHtmlSupportEnabled`; the configuration reference key is the former. Do not copy the blog spelling.

## biome.json

Flag off; Biome's formatter disabled for component files so it does not fight prettier; the four false-positive-prone rules turned off for component files because, with the flag off, Biome sees only a fragment of those files.

```json
{
  "$schema": "https://biomejs.dev/schemas/2.5.0/schema.json",
  "files": {
    "ignoreUnknown": true,
    "includes": ["**", "!**/dist", "!**/.astro", "!**/node_modules"]
  },
  "assist": { "actions": { "source": { "organizeImports": "on" } } },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 100
  },
  "javascript": { "formatter": { "quoteStyle": "single" } },
  "overrides": [
    {
      "includes": ["**/*.astro", "**/*.svelte", "**/*.vue"],
      "formatter": { "enabled": false },
      "linter": {
        "rules": {
          "style": { "useConst": "off", "useImportType": "off" },
          "correctness": { "noUnusedVariables": "off", "noUnusedImports": "off" }
        }
      }
    }
  ]
}
```

The `!**/.astro` exclude is the generated `.astro/` cache directory, not `.astro` files (which the override targets). `formatter.enabled: false` for the component globs is the load-bearing line: if Biome formats `.astro` and prettier also formats it, `biome ci` fails on every template prettier touched. Biome still lints the extracted frontmatter JS in `.astro` (with the four noisy rules off), and `organizeImports` still sorts those imports, which prettier does not do, so the two tools do not overlap.

## prettier for templates

```json
// .prettierrc.json
{
  "plugins": ["prettier-plugin-astro"],
  "overrides": [
    { "files": "*.astro", "options": { "parser": "astro" } }
  ]
}
```

```
# .prettierignore
dist
.astro
node_modules
```

prettier runs only on `*.astro` in practice (Biome owns everything else), so a tight `format` script targets just those files.

## biome ci behavior

`biome ci` is the CI-purposed command: it runs the formatter check, the linter, and import sorting in one pass. It has no `--write` or `--fix`; it only checks and reports. It exits non-zero on any error-level diagnostic, which fails the job. It respects VCS integration with `--changed` (a remote repo has no concept of staged files).

## GitHub reporter

Pass `--reporter=github` for inline GitHub Actions annotations (`::error file=...,line=...::`). Biome does not auto-detect the Actions environment to enable this despite the docs implying it does, so the flag is required. Biome 2.4 and later support multiple reporters and `--reporter-file`, so you can keep a human-readable summary in the log and write a machine file at once:

```bash
biome ci --reporter=github --reporter-file=biome-report.txt
```

## package.json scripts

```json
{
  "scripts": {
    "lint": "biome check",
    "format": "biome format --write && prettier --write \"**/*.astro\"",
    "ci:biome": "biome ci --reporter=github"
  }
}
```

Pin Biome exactly: `npm install --save-dev --save-exact @biomejs/biome@2.5.0`. The CI step runs `npx @biomejs/biome ci --reporter=github` against that pinned dependency, so no version drift. The biomejs/setup-biome@v2 action is an alternative installer that reads the pinned version from the lockfile; the npx-against-pinned-devDep path is simpler and needs no extra action.

## The experimental-flag decision

| Choice | Trade |
|---|---|
| flag off plus prettier-plugin-astro (recommended) | deterministic template formatting with a mature tool; two formatters in the toolchain |
| flag on | single tool; experimental template formatting with breaking diffs that does not match prettier |
| flag off, no prettier | single tool; `.astro` templates are not auto-formatted at all |

Ship flag off plus prettier. Revisit when Biome marks HTML formatting stable: at that point enable `html.experimentalFullSupportEnabled`, drop prettier-plugin-astro, and remove the `formatter.enabled: false` override, returning to Biome-only. Treat the Biome stable-HTML release as the review-gate trigger.

## Stack-canon exception

The single-tool-per-function rule names Biome as the lint and format tool. Adding prettier-plugin-astro for `.astro` template formatting is a deliberate, narrow exception: it exists only because Biome's template formatter is not yet stable, it is scoped to template formatting alone, and it is removed the moment Biome's HTML support stabilizes. Record it as a documented exception, not a second standing tool.

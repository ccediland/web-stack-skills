---
title: web-stack-skills — RESIDENT (working doc / home base)
updated: 2026-06-17
repo: ccediland/web-stack-skills (público, MIT)
status: Fase 0 completada · astro-css-tokens turn 1 (scoping) hecho — 0/7 redactadas — siguiente = astro-css-tokens turn 2 (pre-research)
---

# web-stack-skills — RESIDENT

Documento vivo y **home base** del proyecto: fuente de verdad única. Reemplaza al plan-de-ejecución original. Contiene qué es, el stack, las 7 skills y sus seeds, la cadencia de autoría, las reglas operativas, las decisiones, los descubrimientos del proceso, el estado y el log de sesiones. Quien lo lea queda al día para continuar.

## 1. Qué es

Un **marketplace de plugins de Claude Code**, público y genérico (MIT), que entrega **7 skills** que codifican un stack web premium de alto rendimiento ("lo mejor de todos los mundos"). Reusable en cualquier proyecto: un proyecto origen fue el primer consumidor, **no** el alcance. Cero contenido específico de proyecto, cero marca, cero secretos. Una 8ª skill queda **diferida** como skeleton.

Meta: parándote en cualquier proyecto nuevo, levantar la misma arquitectura sin volver a descubrir los filos.

## 2. El stack (una herramienta por trabajo)

Astro 6 · Cloudflare Workers Static Assets · Tailwind v4 + Style Dictionary · GSAP + CSS scroll-driven + Motion · OGL · Rive · schema-dts + @astrojs/sitemap + llms.txt · CSP nativo · Lighthouse CI + Biome.

## 3. Las 7 skills + 1 diferida

Orden = fundación → visuales. El seed completo de cada una (veredicto, pins, gotchas, outline) vive en su `SKILL.md`; aquí va el resumen.

| # | Skill | Capa | Veredicto (una línea) |
|---|---|---|---|
| 1 | `astro-css-tokens` | fundación | Tailwind v4 `@theme` + Style Dictionary — `tokens.json` (DTCG) a 2 consumidores (CSS vars + `@theme`), sin lock-in |
| 2 | `web-security-headers` | fundación | CSP nativo Astro 6 (`csp:true`) + `_headers` en Workers + `.assetsignore` + SRI |
| 3 | `perf-ci-gates` | fundación | Lighthouse CI (`budget.json`) + Biome, en GitHub Actions |
| 4 | `seo-aeo-schema` | fundación | `schema-dts` `@graph` tipado (`@id` cruzados) + `@astrojs/sitemap` + `llms.txt` |
| 5 | `motion-system` | visuales | 3 motores por trabajo — GSAP (scroll cinemático), CSS scroll-driven (reveals), Motion `useAnimate` mini (islas React) |
| 6 | `webgl-atmosfera` | visuales | OGL + shader GLSL custom para atmósfera de hero (lazy + fallback); WebGPU descartado |
| 7 | `signature-anim` | visuales | Rive (state machine) para un momento interactivo bespoke |

**Diferida — `stack-integration-playbook`** (en `deferred/`, fuera de `skills/`): documenta cómo componen todos los elementos del sitio entre sí y cómo el sitio se conecta al resto del stack más allá de la web (Supabase, otros repos, Cloudflare, GitHub Actions, Infisical, Google Workspace, comms, pagos, catálogos, ads, social, analítica). Se llena "mucho después" con lecciones de campo. Excluida del plugin instalable hasta tener sustancia.

### Pins (2026-06-16 — re-verificar en el research de cada skill)

| Skill | Pins |
|---|---|
| astro-css-tokens | `tailwindcss@4.3.1`, `@tailwindcss/vite@4.3.1`, `style-dictionary` v4 |
| web-security-headers | `astro@6.4.7`, `@astrojs/cloudflare@13.x` |
| perf-ci-gates | `@lhci/cli@0.15.1` (LH 12.6.1), `@biomejs/biome@2.5.0`, Node 22.12+ |
| seo-aeo-schema | `schema-dts`, `@astrojs/sitemap` |
| motion-system | `gsap@3.15.0`, `@gsap/react@2.1.x`, `motion@12.40.0` |
| webgl-atmosfera | `ogl@1.0.11` (exacto; dep estancada — vendorizar) |
| signature-anim | `@rive-app/canvas@2.37.6` (o `@rive-app/webgl2@2.37.8`); NO `@rive-app/webgl` (deprecado) |

Gotchas y outline detallados — en cada `skills/<nombre>/SKILL.md`.

## 4. Estructura del repo

    .claude-plugin/
      marketplace.json     -> 1 plugin: web-stack (source ".")
      plugin.json          -> manifiesto + "skills":[ las 7, rutas relativas ]
    skills/
      astro-css-tokens/SKILL.md
      web-security-headers/SKILL.md
      perf-ci-gates/SKILL.md
      seo-aeo-schema/SKILL.md
      motion-system/SKILL.md
      webgl-atmosfera/SKILL.md
      signature-anim/SKILL.md
    deferred/
      stack-integration-playbook/SKILL.md   (excluido del plugin)
    README.md  LICENSE  RESIDENT.md

Regla: skills bajo `skills/<nombre>/` (nombre de carpeta = nombre de la skill) **y** registradas en `plugin.json`. La diferida vive en `deferred/` = imposible de shippear hasta tener sustancia.

## 5. Cadencia de autoría (5 turnos por skill, ~1 skill por chat)

| Turno | Qué |
|---|---|
| 1 | Entendimiento / scoping — delimitar cobertura, proponer estructura del bundle, enlistar lo que resolver en research |
| 2 | Pre-research — skill `pre-research` (web_search + pre-brief) |
| 3 | Research — Research mode + Context7 (`resolve-library-id` → `query-docs`); **re-verificar versiones** |
| 4 | Cierre / decisiones / preguntas |
| 5 | Build — autoría del bundle → `quick_validate.py` → `package_skill.py` → `.skill` → commit. Al cierre: actualizar este RESIDENT + hand-off al siguiente chat |

## 6. Reglas operativas

- **GitHub = TODO por Composio** (Git Data API vía `proxy_execute` en el workbench). **Claude Code queda reservado estrictamente para el build final** (cuando estén las 7 literal). No abrir Claude Code antes — ni para probar instalación.
- **Un plugin** (`web-stack`) agrupa las 7. Instalación: `/plugin marketplace add ccediland/web-stack-skills` luego `/plugin install web-stack@web-stack-skills`.
- Gobierno: **skill-author** = autoridad de arquitectura; **skill-creator** (`/mnt/skills/examples/skill-creator`, con `quick_validate.py` + `package_skill.py`) = validar/empacar; **Context7** + docs oficiales = contenido actual.
- Naming de skills: kebab-case, ≤64, sin la palabra "claude". Description ≤1024, sin `<` ni `>`, sin dos-puntos-espacio a media cadena (rompe YAML — usar guión largo).
- Pins del 2026-06-16 — re-verificar en turns 2/3 de cada skill.
- `SKILL.md` = veredicto + receta; `references/` = configs/plantillas/gotchas (progressive disclosure).

## 7. Decisiones

- Público + genérico + MIT. Un repo-marketplace, un plugin que agrupa las 7.
- 8ª skill diferida como skeleton; excluida del instalable hasta tener sustancia.
- Skills bajo `skills/<nombre>/`; manifiesto `plugin.json` agregado; skills registradas explícitamente en `plugin.json`; diferida fuera de `skills/`.

## 8. Descubrimientos del proceso (2026-06-16/17)

- **Layout de plugin** (verificado vs docs oficiales): skills van bajo `skills/<n>/SKILL.md` (no en raíz); hace falta `.claude-plugin/plugin.json`; las skills deben **registrarse** en `plugin.json` vía `"skills":["./skills/..."]` — el auto-discovery solo no basta.
- **Quirk de GitHub** (verificado en ejecución): en repo recién creado **vacío**, la Git Data API (`git/trees`) da 409 "Git Repository is empty" → bootstrapear con un commit inicial (Contents API) y luego reemplazar el árbol completo.
- Poner la 8ª en `deferred/` (fuera de `skills/`) la excluye **estructuralmente** — más limpio que listas de exclusión en el manifiesto.

## 9. Estado

- Fase 0 (scaffold) — completada.
- astro-css-tokens turn 1 (Entendimiento) — hecho (alcance + estructura del bundle + checklist de research; forks abiertos para turn 4).
- 0/7 skills redactadas (8 skeletons en su sitio, frontmatter válido).
- Siguiente — `astro-css-tokens`, turn 2 (Pre-research).

## 10. Roadmap

- **Fase 0** — scaffold del marketplace. Hecha.
- **Skills 1–7** — autoría por la cadencia de 5 turnos, orden fundación → visuales. (0/7)
- **Build final (Claude Code)** — `quick_validate` + `package` de las 7, prueba de `marketplace add` / `install` + triggering; después empezar a llenar la skill diferida con lecciones de campo.

## 11. Log de sesiones

### 2026-06-16/17 — Fase 0 (scaffold) — hecha

- Identidad confirmada (`ccediland`). Repo público creado.
- 13 archivos pusheados por Composio (Git Data API), commit scaffold `a4bd7f0`; bootstrap previo por el quirk de repo vacío.
- `plugin.json` parcheado para registrar las 7 (`"skills":[...]`), commit `b24c124`.
- Frontmatter de los 8 SKILL.md validado inline (el `quick_validate.py` real corre en el build).
- Descubrimientos en §8.
- Siguiente: `astro-css-tokens` turn 1 en chat nuevo.

### 2026-06-17 — astro-css-tokens · turn 1 (Entendimiento) — hecho

- Alcance delimitado. Dentro: tokens.json DTCG (fuente única) -> SD v4 -> 2 salidas (CSS vars planas + bloque @theme) · 3 capas base/semantic/component vía alias DTCG · OKLCH · wiring Astro (global.css, @tailwindcss/vite, dónde corre el build) · config SD que emite ambas salidas · gotchas. Parcial: dark/multi-tema (solo la costura), dims no-color (ejemplo mínimo). Fuera: estilo a nivel componente, diseño de paleta, stacks no-Astro, migración.
- Forks abiertos (resolver turn 4): (1) dark/multi-tema — costura inline vs build multi-tema de SD como reference; (2) ejemplo solo color vs +spacing/tipografía; (3) ejemplo como código fenced en references vs archivos en assets/. Sesgos: costura inline · sí dim extra mínima · fenced en references.
- Estructura del bundle propuesta: SKILL.md (veredicto + receta + gotchas + punteros) + references/ {style-dictionary-config, tokens-example, astro-tailwind-wiring, color-oklch, gotchas}. Calls menores: gotchas inline si ≤~8; posible fusión config+wiring en build-setup.md.
- Checklist de research (turns 2/3): re-verificar pins (tailwindcss/@tailwindcss/vite 4.3.1?, style-dictionary 4.x, astro 6.x) · soporte DTCG en SD v4 (flag usesDtcg/preprocessor, sintaxis $value/$type, alias {group.token}) · custom format SD->@theme (SD v4 no lo trae de fábrica) · @theme vs @theme inline (sostiene la tesis no-lock-in) · OKLCH default + fallback · @reference "tailwindcss" para @apply scoped · @astrojs/tailwind muerto en v4 · dónde corre style-dictionary build (sesgo: prebuild script, native-first).
- CASO CONTRARIO a defender en research: Tailwind v4 @theme ya genera CSS vars solo -> ¿para qué SD? La skill debe justificar SD por multi-output + transforms + portabilidad DTCG, o se reescribe el veredicto a @theme + CSS vars hand-authored.
- Siguiente: astro-css-tokens turn 2 (Pre-research) en chat nuevo.

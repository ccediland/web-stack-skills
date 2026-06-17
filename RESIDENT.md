---
title: web-stack-skills — RESIDENT (working doc / home base)
updated: 2026-06-17
repo: ccediland/web-stack-skills (público, MIT)
status: web-security-headers turn 1 (scoping) HECHO — 1/7 skills redactadas — siguiente = web-security-headers turn 2 (pre-research)
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
| 1 | `astro-css-tokens` | fundación | DTCG `tokens.json` → Style Dictionary **v5** → capa `:root --ds-*` (fuente de verdad) + bridge `@theme inline`; OKLCH sin fallback; en Astro 6 vía `@tailwindcss/postcss` (vite roto, #16542) |
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
| astro-css-tokens | `astro@6.4.7`, `tailwindcss@4.3.1`, `@tailwindcss/postcss@4.3.0` (NO `@tailwindcss/vite` — #16542), `style-dictionary@5.4.4` (v5, no v4), Node ≥22.12 — verificar en build |
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

### 2026-06-17 — research astro-css-tokens (verificado, fuente primaria)

- **Style Dictionary v5** es el actual (npm 5.4.4); drop-in desde v4 (breaking budget mínimo: refs no apuntan a hojas no-token ni sufijo `.value`; delimitadores `{…}` fijos; Node ≥22). Toda la API que usamos (`new StyleDictionary`, `registerFormat`, `usesDtcg`, `outputReferences`, `css/variables`) sin cambio.
- **Astro 6 × rolldown-vite rompe `@tailwindcss/vite`** (#16542 abierto a jun-2026; causa raíz #19802 `aliasOnly:true`). Camino que compila: `@tailwindcss/postcss` + `postcss.config.mjs` (Astro corre PostCSS nativo). Pinear Vite a v6 también, pero pelea con Astro 6.
- **OKLCH `oklch()` = Baseline Widely Available desde 2025-11-09** (Chrome/Edge 111, Firefox 113, Safari 15.4). Se shippea sin fallback. Tailwind v4 trae su paleta default en OKLCH.
- **El transform SD `color/css` convierte OKLCH a hex/rgb** (Color.js, gamut-map a sRGB). Para preservar OKLCH: transforms solo-nombre (`attribute/cti`, `name/kebab`), sin grupo `css`. (SD v5.3+ trae `color/oklch` pero opera sobre color DTCG estructurado, no string plano.)
- **`@theme inline`** no emite var global; la utility compila a `var(--ref)` — es el mecanismo del bridge. `@theme` plano sí emite var global; `@theme static` fuerza emitir todas las vars aunque no se usen.
- **DTCG composite split bug** (#1398/#1494): `$value` objeto (`{value, unit}`) emite vars partidas `-value`/`-unit`. Workaround: `$value` string plano.
- **DTCG estabilizó 2025.10** (28-oct-2025, primera versión estable); SD da soporte.
- Ref de implementación SD→`@theme`: `tokens-studio/sd-tailwindv4` (~15★, README admite patrones no-DTCG; minar por forma, no como dependencia).

## 9. Estado

- Fase 0 (scaffold) — completada.
- astro-css-tokens turns 1–4 — hechos (scoping, pre-research, research verificado, decisiones cerradas). Veredicto y pins reescritos arriba (§3).
- 1/7 skills redactadas (`astro-css-tokens` — turn 5 completo).
- `web-security-headers` — turn 1 (scoping) hecho; alcance, estructura de archivos, checklist de research y forks definidos (log §11).
- Siguiente — `web-security-headers`, turn 2 (Pre-research).

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

### 2026-06-17 — astro-css-tokens · turns 2–4 (pre-research · research · decisiones) — hechos

**Turn 2 (pre-research):** pre-brief armado. Caso contrario resuelto — SD se justifica solo con ≥2 consumidores / portabilidad; si no, `@theme` nativo basta. Detectado el build-break de Astro 6.

**Turn 3 (research, verificado fuente primaria):** seis subtasks cerrados → descubrimientos en §8.

**Turn 4 — decisiones cerradas (spec para el build):**
- Compilador: **Style Dictionary v5** (5.4.4), drop-in desde v4.
- Install Tailwind en Astro 6: **`@tailwindcss/postcss`** + `postcss.config.mjs`; NO `@tailwindcss/vite` (#16542 abierto). Review gate: volver a vite cuando #16542 cierre + repro verde.
- Bridge: **`@theme inline`** sobre capa `:root --ds-*` (fuente de verdad CSS pura); no `@theme` plano. Un solo override point; sirve utilities + CSS a mano + 2º consumidor.
- Color: OKLCH **sin fallback** (Baseline 2025-11-09). Transforms SD solo-nombre; NUNCA `color/css`.
- Tokens DTCG: `$value` **string plano** (evita split #1398/#1494). Capas base→semantic→component, alias `{…}`, `outputReferences:true`, `usesDtcg:true`.
- Build SD: **prebuild script** (`node build.mjs`), native-first.
- Dark mode: solo la costura — `@custom-variant dark (&:where([data-theme="dark"], …))` + `[data-theme="dark"]{ --ds-color-*: oklch(oscuro) }`.

**Arquitectura fijada:** `tokens.json` → SD v5 → `tokens.css` (`:root --ds-*`, css/variables) + `theme.css` (`@theme inline { --color-*: var(--ds-color-*) }`). Utilities/CSS/2º consumidor leen `--ds-*`.

**Veredicto reescrito** (reemplaza al del seed): un `tokens.json` DTCG → SD v5 → capa `:root --ds-*` (fuente de verdad framework-independiente) + bridge `@theme inline` delgado que la referencia. Justificado con ≥2 consumidores / portabilidad; si no, `@theme` solo basta. En Astro 6 vía `@tailwindcss/postcss`.

**Forks turn 1 cerrados:** dark = costura inline (sin multi-tema build); ejemplo = color+spacing+font+radius mínimos; ejemplo como código fenced en `references/` (no `assets/`).

**Estructura del bundle (sin cambio):** SKILL.md + references/{style-dictionary-config, tokens-example, astro-tailwind-wiring, color-oklch, gotchas}.

**Pendiente de verificar en el build:** patches exactos en npm (`tailwindcss`/`@tailwindcss/postcss` mostraron 4.3.1 vs 4.3.0; SD npm 5.4.4 vs tag GitHub 5.4.2).

Siguiente: astro-css-tokens turn 5 (Build).

### 2026-06-17 — astro-css-tokens · turn 5 (Build) — HECHO · commit 9b182b0

Bundle autoreado y commiteado en un commit atómico (Git Data API):

- `skills/astro-css-tokens/SKILL.md` (108 líneas) — veredicto, diagrama de arquitectura, tabla de versiones pinadas, 3 capas, setup en orden (5 pasos), gotchas, limitaciones, pointers a references.
- `skills/astro-css-tokens/references/style-dictionary-config.md` (114 líneas) — `build.mjs` verificado completo: `TW_NAMESPACE`, `UTILITY_LAYERS`, formato custom `css/tailwind-theme`, config SD v5, tabla de partes, hooks `package.json`, extensión, gotchas SD.
- `skills/astro-css-tokens/references/tokens-example.md` (128 líneas) — `tokens/base.json` + `tokens/semantic.json` + `tokens/component.json` en DTCG verificado; reglas de autoría; output compilado `tokens.css` + `theme.css`; gotcha `$value` objeto.
- `skills/astro-css-tokens/references/astro-tailwind-wiring.md` (99 líneas) — por qué `@tailwindcss/postcss`, `postcss.config.mjs`, orden de imports `global.css`, script no-flash `is:inline`, `@reference "tailwindcss"` para `@apply` en scoped styles, switch-back trigger #16542.
- `skills/astro-css-tokens/references/color-oklch.md` (39 líneas) — OKLCH perceptualmente uniforme, Baseline 2025-11-09 (sin fallback), preservación en SD (transforms solo-nombre), escape hatch legacy.

Validado: `quick_validate PASS` | Empaquetado: `astro-css-tokens.skill` 8.7 K.
Commit: [9b182b0](https://github.com/ccediland/web-stack-skills/commit/9b182b00b3595d2b755cbf27e485e90870aa7c56)

Siguiente: web-security-headers turn 1 (Scoping).

### 2026-06-17 — web-security-headers · turn 1 (Scoping) — HECHO

**Reframe que reordena el bundle** (verificado, blog Astro 6 + config reference): el CSP de Astro 6 es **estable** (salió de `experimental:`), **basado en hashes**, e inyectado como `<meta http-equiv="content-security-policy">` en el HTML — **no** como response header. Hashea scripts/estilos bundleados (incl. cargados dinámicamente); funciona estático y SSR; imágenes responsivas ya cubiertas. Consecuencias duras: (a) el CSP de Astro **NO pasa por `_headers`**; (b) `<meta>` **ignora** `frame-ancestors`, `report-to`/`report-uri` y `sandbox` → esas directivas obligan un **header real**; (c) el **nonce** deja de ser el eje y pasa a escape hatch de nicho (Astro es hash-first; issue #377: nonces en `<style>` se eliminan).

**Alcance IN:** CSP nativo Astro 6 (`security.csp`, hashes, cómo añadir hashes externos); set de security headers vía Cloudflare `_headers` (HSTS, X-Frame-Options + `frame-ancestors`, X-Content-Type-Options, Referrer-Policy, Permissions-Policy, Cross-Origin-* según necesidad); el split `<meta>` vs header real; middleware de Astro (`src/middleware.ts`) como escape hatch dinámico (headers por-ruta, CSP como header real, ruta de nonce); SRI para externos; `.assetsignore` (higiene Workers Static Assets); baseline copy-paste + verificación (curl, securityheaders.com, CSP evaluator, consola).

**Alcance OUT (pointer, sin desarrollo):** CORS de APIs / WAF / rate-limit / bots; cookies (SameSite/Secure/HttpOnly, capa app); Cache-Control / headers de perf (→ `perf-ci-gates` #3, fork F6); backend de reportes CSP; nonce como vía SSR principal.

**Estructura preliminar** (granularidad final en build): `SKILL.md` + `references/`: `csp-astro-native.md`, `cloudflare-headers.md`, `middleware-and-nonce.md`, `header-inventory.md`, `sri-and-verification.md` (5 refs tentativas).

**Checklist de research (turn 2→3):** 1) config exacta `security.csp` estable vs flag viejo, ¿opción de header real en adapter Cloudflare? 2) `@astrojs/cloudflare@13.x` / Workers Static Assets — ¿emite headers reales o siempre meta-CSP + `_headers`? 3) directivas ignoradas en `<meta>` en navegadores actuales; 4) sintaxis/precedencia/límites de `_headers` en WSA, confirmar static-only; 5) `.assetsignore` propósito exacto con adapter; 6) ¿Astro 6 genera SRI auto o manual?; 7) ruta de nonce por middleware (¿soportado o hash-only?); 8) HSTS preload + conflicto con HSTS de dashboard Cloudflare; 9) baseline Permissions-Policy; 10) COOP/COEP/CORP (COEP rompe embeds → interacción #6 OGL/#7 Rive); 11) ¿Cloudflare inyecta headers por default que choquen?; 12) re-verificar pins `astro@6.4.7`, `@astrojs/cloudflare@13.x`.

**Forks abiertos (turn 4):** F1 entrega CSP (`<meta>` hashes default vs header real) → default meta + header solo para frame-ancestors/report-to. F2 headers no-CSP (`_headers` vs middleware) → `_headers` canónico, middleware solo dinámico. F3 nonce (documentar escape hatch vs hash-only) → hash-only + nota. F4 COOP/COEP/CORP (default vs opt-in) → opt-in con nota. F5 SRI (paso doc vs pointer) → depende de research #6. F6 cache headers (incluir en `_headers` vs ceder a #3) → pointer a #3. F7 granularidad refs (4 vs 5). F8 Pages vs Workers Static Assets → target WSA (RESIDENT), notar compat Pages.

**Discrepancias reconciliadas:** seed de chat decía "+ `.astro` middleware"; veredicto RESIDENT §3 decía "+ `.assetsignore` + SRI" → alcance cubre la **unión** (middleware + .assetsignore + SRI). Pages (seed) vs Workers Static Assets (RESIDENT) → canónico WSA, compat Pages notada (F8).

**Descubrimiento de proceso:** `raw.githubusercontent.com` sirve copia cacheada/vieja del RESIDENT (mostró Fase 0 / 0/7 mientras el HEAD real iba 1/7). Para leer el estado real del RESIDENT, usar **Contents API vía Composio** (`proxy_execute GET .../contents/RESIDENT.md` → content+sha), no el raw. Sandbox suffix activo este chat: `axpx`.

Siguiente: web-security-headers turn 2 (Pre-research).

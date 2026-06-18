---
title: web-stack-skills — RESIDENT (working doc / home base)
updated: 2026-06-17
repo: ccediland/web-stack-skills (público, MIT)
status: seo-aeo-schema turn 5 (build) HECHO — SKILL.md + 5 refs commiteadas (0b55775), fuente byte-verificada — 4/7 redactadas — siguiente = motion-system (#5) turn 1 (Scoping)
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
| 2 | `web-security-headers` | fundación | CSP nativo Astro 6 (`security.csp`, hash-based, `<meta>` en estático — CF@13 no soporta `staticHeaders`) + `public/_headers` para HSTS/frame-ancestors/Referrer/Permissions/COOP-CORP (lo que `<meta>` ignora) + middleware solo SSR; SRI manual/astro-shield solo cross-origin; nonce = hash-only |
| 3 | `perf-ci-gates` | fundación | Dos puertas en GitHub Actions — (a) LHCI vía `treosh/lighthouse-ci-action@v12`, `staticDistDir ./dist/client`, preset `no-pwa` + floors (perf 0.9 err / a11y 0.95 err; best-practices+seo warn) + budgets de métrica (LCP/TBT/CLS err) + `budget.json` (KB), INP→TBT proxy, mediana de 5, `temporary-public-storage`+artifacts; (b) Biome `biome ci --reporter=github`, flag `.astro` OFF + `prettier-plugin-astro` (template) + 4 overrides. CWV oficiales SIN cambio (LCP 2.5s/INP 200ms/CLS 0.1) — claim 2.0s FALSO. Gate-only (caching = nota de una pantalla) |
| 4 | `seo-aeo-schema` | fundación | Capa de descubribilidad machine-readable. `schema-dts@2.0.0` (TS-only) → un @graph JSON-LD centralizado cross-@id (Organization↔WebSite[+SearchAction]↔WebPage↔BreadcrumbList↔entidad) emitido inline vía render util con escape; el bloque `application/ld+json` es data block EXENTO del CSP de #2 (no ejecuta → script-src no aplica → no necesita hash). `@astrojs/sitemap@3.7.3` opinionado (solo rutas prerendered; SSR vía endpoint propio; lastmod por serialize). `robots.txt` con tokens AI-crawler verificados (training opt-out = decisión documentada; Googlebot siempre allow; disallow≠fuera-de-respuestas). `llms.txt` como archivo B2A/agent-readiness (Lighthouse 13.3 lo audita) vía endpoint prerendered — NO palanca SEO (Google no lo consume). Componente meta/head typed (canonical/OG/Twitter). Validación = schema-dts compile-time + checks manuales (Rich Results Test + validator.schema.org) + script custom exportado al gate de #3. JSON-LD NO da inclusión en AI Overviews (Google: no special schema) — valor = entity recognition + rich results sobrevivientes. |
| 5 | `motion-system` | visuales | 3 motores por trabajo — GSAP (scroll cinemático), CSS scroll-driven (reveals), Motion `useAnimate` mini (islas React) |
| 6 | `webgl-atmosfera` | visuales | OGL + shader GLSL custom para atmósfera de hero (lazy + fallback); WebGPU descartado |
| 7 | `signature-anim` | visuales | Rive (state machine) para un momento interactivo bespoke |

**Diferida — `stack-integration-playbook`** (en `deferred/`, fuera de `skills/`): documenta cómo componen todos los elementos del sitio entre sí y cómo el sitio se conecta al resto del stack más allá de la web (Supabase, otros repos, Cloudflare, GitHub Actions, Infisical, Google Workspace, comms, pagos, catálogos, ads, social, analítica). Se llena "mucho después" con lecciones de campo. Excluida del plugin instalable hasta tener sustancia.

**Planeada (post-7) — `cms-self-edit`** (placeholder; nombre final se fija en el turno de selección): capacidad de **self-edit** para clientes (CMS headless). Cadencia especial — primero un **turno de selección de herramienta** (Sanity/Storyblok/Tina/Keystatic u otro) porque hoy NO hay CMS web en el stack-canon (gap a resolver y registrar en `stack-canon-argos`), luego la cadencia normal de 5 turnos para documentar el stack elegido. Scaffold al repo (carpeta/nombre/`plugin.json`) DIFERIDO hasta llegar a esa etapa.

### Pins (2026-06-16 — re-verificar en el research de cada skill)

| Skill | Pins |
|---|---|
| astro-css-tokens | `astro@6.4.7`, `tailwindcss@4.3.1`, `@tailwindcss/postcss@4.3.0` (NO `@tailwindcss/vite` — #16542), `style-dictionary@5.4.4` (v5, no v4), Node ≥22.12 — verificar en build |
| web-security-headers | `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`; SRI opc. `@kindspells/astro-shield@1.7.1` (≥1.3.2 por CVE-2024-30250). Review-gate: `staticHeaders` en cada minor de `@astrojs/cloudflare`; CSP no testeable en `dev` (build+preview); watch Astro 7 alpha |
| perf-ci-gates | `@lhci/cli@0.15.1` (LH 12.6.1), `treosh/lighthouse-ci-action@v12`, `@biomejs/biome@2.5.0` (`--save-exact`; conservador `2.4.16`), `prettier-plugin-astro` (solo format de template `.astro`), `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`, **Node 22** (piso Astro 6). Review-gate: thresholds CWV contra web.dev/Search Central (rechazar claim 2.0s); flag `html.experimentalFullSupportEnabled` ON cuando Biome marque HTML estable; LH13 bloqueado de LHCI por Node 22.19+. Verificado 2026-06-17 |
| seo-aeo-schema | `schema-dts@2.0.0` (devDep, TS-only, Schema.org v30; NO valida @id cross-refs — son strings; breaking vs 1.x: Role no-recursivo, Quantity core DataType, dep `schema-dts-lib`), `@astrojs/sitemap@3.7.3` (regresión 3.7.1 #15894 fija; NO ve rutas SSR; bug #16838 lastmod ausente en sitemap-index). Baseline `astro@6.4.7`, `@astrojs/cloudflare@13.7.0` (sin staticHeaders → CSP por `<meta>`; text endpoints requieren `prerender=true`; assets en `dist/client`), Node 22. Review-gates: profundidad de @graph → `…Leaf`/`MergeLeafTypes` (no bajar a 1.1.5); deprecación FAQ (RRT pierde FAQ jun-2026, SC API ago-2026); audit llms.txt de LH13 aún NO en LHCI (#3 corre LH12) |
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
- `astro-css-tokens` y `web-security-headers` — completas, 5/5 turns cada una. 2/7 redactadas. Fuentes de web-security-headers en commit 4f37a05 (SKILL.md + 5 references).
- `perf-ci-gates` (#3) turn 1 (Scoping) — HECHO. Alcance = gate de CI de dos puertas (LHCI + Biome en GitHub Actions). Tool-selection ya lockeado por stack-canon (no requiere turno de selección). 10 forks abiertos (F1–F10), caso-contrario nombrado, checklist de research de 9 puntos.
- `perf-ci-gates` (#3) turn 2 (Pre-research) — HECHO. `pre-research` corrida: 8 búsquedas (entre turns 1–2) + fuentes primarias; pre-brief de 8 subtasks armado para Research mode. Hallazgos clave a verificar en turn 3 (abajo en §11).
- `perf-ci-gates` (#3) turns 3–4 (Research + Decisions) — HECHOS. Research mode entregó reporte verificado contra fuentes primarias. Forks F1–F10 lockeados (detalle en §11). 2 reversals vs leans previos: F4 (raw autorun → treosh action) y F5 (flag `.astro` ON → OFF + prettier-plugin-astro). Caso contrario derrotado; claim CWV 2.0s confirmado FALSO.
- `perf-ci-gates` (#3) turn 5 (Build) — HECHO. Bundle (SKILL.md + 4 refs) autorado, `quick_validate` PASS (description 998/1024, sin `<`/`>`, sin `: `, body house-style limpio), `.skill` empacado (15.3 KB), fuente commiteada atómica (Git Data API) en `23d8388f`. Stub sobrescrito. (Turn 5 se reanudó tras un corte; el bundle del run cortado persistió en el contenedor de code-exec, se verificó contra los forks lockeados y se re-validó antes de commitear.)
- `seo-aeo-schema` (#4, fundación) turn 1 (Scoping) — HECHO. Alcance IN/PARTIAL/OUT; 10 forks (F1–F10) con leans; caso contrario doble; seam CSP×JSON-LD (×#2) load-bearing; checklist de research. Tool-selection NO requerido.
- `seo-aeo-schema` (#4) turn 2 (Pre-research) — HECHO. Skill `pre-research` corrida; F1 resuelto IN; pre-brief de 8 subtasks entregado.
- `seo-aeo-schema` (#4) turn 3 (Research) — HECHO. Research mode entregó reporte verificado (fuentes primarias). De-risking clave: el seam CSP×JSON-LD (×#2) se RESUELVE como NO-ISSUE — `ld+json` es data block exento de script-src, no necesita hash. schema-dts@2.0.0 vivo (caso contrario primario derrotado) pero NO valida @id cross-refs. Detalle en §11.
- `seo-aeo-schema` (#4) turn 4 (Decisiones) — HECHO. F2–F10 lockeados; §3 veredicto/pins reescritos. Detalle en §11.
- `seo-aeo-schema` (#4) turn 5 (Build) — HECHO. SKILL.md + 5 refs autoradas, validadas (quick_validate OK; description 961 chars; sin `<`/`>`; frontmatter solo {name, description}; sin bold/HR/H4; TOC en los 2 refs >100 líneas), empaquetadas (.skill 15750 B), y commiteadas atómicamente vía Git Data API (commit 0b55775, parent 33cb7cc, autor Carlos; fuente byte-verificada: sha git de cada blob == sha local antes de mover el ref). Skill #4 COMPLETA. Detalle en §11.
- Siguiente — `motion-system` (#5) turn 1 (Scoping).

## 10. Roadmap

- **Fase 0** — scaffold del marketplace. Hecha.
- **Skills 1–7** — autoría por la cadencia de 5 turnos, orden fundación → visuales. (4/7 redactadas; #2 `web-security-headers` 4f37a05; #3 `perf-ci-gates` 23d8388f; #4 `seo-aeo-schema` 0b55775 — COMPLETA; siguiente = #5 `motion-system` turn 1)
- **Skill planeada (post-7) — `cms-self-edit`** (placeholder) — capacidad self-edit/CMS headless; turno de selección de herramienta primero (gap de stack-canon), luego cadencia normal; scaffold al repo diferido hasta llegar.
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

### 2026-06-17 — web-security-headers · turn 2 (Pre-research) — HECHO

Skill `pre-research` corrida: pre-investigación (8 búsquedas + fuentes primarias) + pre-brief para turn 3 (Research mode on). Hallazgos clave (**pre-investigación, a verificar en turn 3**):

- **Entrega del CSP en Astro 6 es por modo de página, automática:** páginas estáticas → `<meta http-equiv="content-security-policy">`; páginas on-demand (SSR) → header `Content-Security-Policy`. No es elección libre (changelog 5.9.3 + blog Astro 6 + config reference).
- **`experimentalStaticHeaders` = Astro Adapter Feature:** si el adapter lo soporta, Astro deja de emitir el `<meta>` en estáticas y entrega los headers vía hook `astro:build:generated` (`experimentalRouteToHeaders`) para que el adapter escriba un archivo de config (p.ej. `_headers`). Fuentes listan Vercel/Netlify/Node con soporte; **Cloudflare NO confirmado** → pivote del research (decide F1/F2).
- **`<meta>` CSP ignora `frame-ancestors`, `report-uri`/`report-to` y `sandbox`** (W3C CSP2 §, CSP3, OWASP, MDN) y no soporta `-Report-Only` → esas directivas + clickjacking obligan **header real** (en `_headers` o middleware). Confirma el split CSP/headers del turn 1.
- **Cloudflare `_headers`:** archivo de texto sin extensión en `public/`, parseado por la plataforma (no servido como asset), aplica a respuestas de assets estáticos; static-only (sin per-request). Quirk conocido de cache en Workers assets (issue withastro/astro #13164).
- **SRI no es nativo en Astro:** lo provee `@kindspells/astro-shield` (SRI on-by-default en estático + CSP headers) o se hace manual; astro-shield tuvo CVE-2024-30250 (parchado 1.3.2) y su rol de CSP ahora **solapa** con el nativo de Astro 6 → su valor único hoy es SRI. Alternativa: `abemedia/astro-static-headers` (captura `Astro.response.headers.set` → escribe `_headers`; requiere `prerenderEnvironment: 'node'` con `@astrojs/cloudflare` v13+). SRI importa sobre todo para recursos cross-origin.

**Subtasks de research (7 — versión completa en el pre-brief del chat):** 1) ¿`@astrojs/cloudflare@13` soporta `experimentalStaticHeaders`?; 2) shape de `security.csp` en `astro@6.4.7` (`directives`, `scriptDirective`/`styleDirective`, `strictDynamic`, hash algo) + runtime API `Astro.csp.*`; 3) `_headers` en Workers Static Assets (sintaxis/matching/precedencia/estado del quirk de cache); 4) `.assetsignore` con cloudflare@13 (propósito, qué listar, ¿lo genera Astro?); 5) SRI nativo vs astro-shield (mantenimiento 2026, overlap con CSP nativo, CVE resuelto, alcance); 6) ruta de nonce para inline de terceros (¿hash-only?) + estado issue #377; 7) baseline 2026 de headers no-CSP (HSTS `includeSubDomains`/`preload`, Permissions-Policy, COOP/COEP/CORP — impacto en embeds de #6 OGL/#7 Rive, Referrer-Policy) + ¿Cloudflare inyecta headers por default que choquen?

**Source priority:** Astro docs oficiales > Cloudflare Developer docs > W3C CSP3 + MDN > OWASP CSP Cheat Sheet > GitHub repos/changelogs (`withastro/astro`, `@astrojs/cloudflare`) > integraciones de terceros (`astro-shield`, `astro-static-headers` — marcar SECONDARY, verificar mantenimiento + CVEs) > blogs técnicos [SECONDARY].

Siguiente: web-security-headers turn 3 (Research — Research mode + Context7; re-verificar pins `astro@6.4.7`, `@astrojs/cloudflare@13.x`).

### 2026-06-17 — web-security-headers · turn 3 (Research) — HECHO · informe completo en chat

Research mode + web/source verification. Informe completo en el chat del turn 3; aquí el esqueleto decisión-relevante (turn 4 reescribe veredicto/pins en §3).

**Pins verificados (npm, 2026-06-17):** `astro@6.4.7` (CSP estable bajo `security.csp`, "Added in astro@6.0.0"); `@astrojs/cloudflare@13.7.0`. Watch: Astro 7 alpha (Vite 8, compilador Rust default) ya tageado — puede cambiar internals de CSP/adapter en un major futuro.

**Pivote (F1/F2) resuelto — `@astrojs/cloudflare@13.7.0` NO soporta `staticHeaders`/`experimentalStaticHeaders`:** verificado contra el source del adapter (la interfaz `Options` y el bloque `adapterFeatures` no lo declaran, y no consume el map `astro:build:generated`/`routeToHeaders`). El core añadió la feature (PR #13926) y la estabilizó como `staticHeaders` (PR #15258, ejemplos Netlify/Node); el blog Astro 5.11 prometió Cloudflare "coming soon… vía `_headers`" pero NO ha shippeado. Consecuencia: en Cloudflare estático el CSP de Astro queda como `<meta>`; no hay path nativo para emitirlo como header a `_headers`. `abemedia/astro-static-headers` sí puede, pero exige `cloudflare({ prerenderEnvironment: 'node' })`.

**`security.csp` (shape estable):** `algorithm` (SHA-256/384/512), `directives[]` (NO admite `script-src`/`style-src` aquí → usar las sub-claves), `scriptDirective`/`styleDirective` cada una con `hashes[]` + `resources[]`, `scriptDirective.strictDynamic`. Runtime API `Astro.csp.insertDirective/insertScriptResource/insertStyleResource`. Recursos externos = hashes manuales. No corre en `dev` (usar `build`+`preview`). Incompatibles: `unsafe-inline`, `<ClientRouter/>` view transitions, Shiki (estilos inline → usar `<Prism/>`). Imágenes responsivas: cubiertas.

**`<meta>` no expresa `frame-ancestors`, `report-uri`, `sandbox` ni report-only** (W3C/MDN; reproducido en Astro #13896) → obligan header real en `_headers`.

**`_headers` (Workers Static Assets):** texto plano en `public/`, aplica SOLO a respuestas de assets estáticos (NO a respuestas del Worker/SSR), mismo formato que Pages. Quirk #13164 (Cache-Control no aplicaba) = `wontfix`/upstream; pero el adapter v13 auto-inyecta Cache-Control immutable para `_astro/*` hasheados. `.assetsignore` (sintaxis .gitignore, raíz de assets) excluye `_worker.js`/`_routes.json`/`_headers` de servirse; lo maneja adapter/wrangler.

**SRI:** no nativo en Astro 6. `@kindspells/astro-shield` 1.7.1 (CVE-2024-30250 CVSS 7.5, parchado en 1.3.2); su rol CSP solapa con el nativo → usarlo SOLO para SRI, sobre todo cross-origin. **Nonce:** hash-only; no hay modo nonce; nonce en `<style>` se elimina (#377) → inline de tercero = agregar su hash en `scriptDirective.hashes`; nonce real exige header desde SSR.

**Baseline no-CSP:** HSTS `max-age=63072000; includeSubDomains` (preload OFF por default — OWASP avisa consecuencias permanentes; en CF preferir HSTS zone-wide en dashboard); `X-Content-Type-Options: nosniff`; `Referrer-Policy: strict-origin-when-cross-origin`; `X-Frame-Options: DENY` + `frame-ancestors 'none'` (en `_headers`); Permissions-Policy apagar features no usados; COOP `same-origin` OK, **COEP OFF por default** (`require-corp` rompe assets cross-origin de OGL #6 / Rive #7), CORP `same-origin`. CF Managed Transform "Add security headers" omite CSP+Permissions-Policy; evitar duplicar headers entre `_headers` y dashboard.

**Recomendaciones F1–F8 (entran a turn 4):** F1 `<meta>` hashes default, sin forzar header en CF; frame-ancestors/report en `_headers`. F2 `_headers` canónico (estático) + middleware solo en rutas SSR. F3 hash-only. F4 COOP on, COEP opt-in. F5 astro-shield ≥1.3.2 solo para SRI cross-origin; si no, manual. F6 caching de assets hasheados lo hace el adapter; Cache-Control custom mínimo/evitar (#13164); diferir perf a otra skill. F7 split de 5 references soportado. F8 Workers Static Assets canónico (Pages removido en v13); `_headers` porta a Pages.

**Caveats:** el veredicto "CF no soporta staticHeaders" se apoya en el source de `main` + ausencia en changelog (no se leyó el changelog línea por línea) → si un release futuro lo agrega, F1 cambia. Algunos blogs 2026 confunden entrega por `<meta>` (sí funciona) con header (no) → poco fiables. #13164 verificar empíricamente en deploy real. CSP no testeable en `dev`.

Siguiente: web-security-headers turn 4 (Decisiones — cerrar F1–F8, reescribir §3).

### 2026-06-17 — Revisión externa (dev front-end senior) — REGISTRADO (filtrado)

Revisión externa del stack como artefacto terminado y como método. Se registran SOLO los takeaways genéricos/de proceso; este RESIDENT es público y el repo es charter "cero secretos / genérico", así que la tesis comercial/GTM se mantuvo deliberadamente FUERA (ver más abajo).

Adoptado (genérico, rationale de diseño):
- **El activo durable es el MÉTODO, no las 7 skills.** El motor portable (este RESIDENT + cadencia 5-turn + caso-contrario + source-priority + log) sobrevive a cualquier stack/dominio; las skills son perecederas y reconstruibles desde docs (con costo de verificación no trivial — cf. el pivote del turn 3, que salió del source, no de docs). El repo se organiza alrededor del motor a propósito.
- **La skill diferida `stack-integration-playbook` concentra el valor irremplazable** (lecciones de composición de campo), no las 7 fundación/visuales. Justifica subir su prioridad de alimentación cuando haya sustancia de campo.
- **Disciplina de perecederos:** cada pin duro y cada workaround por issue-number lleva review-gate (como #16542 en skill #1), no solo "re-verificar". Solo + sin auto-update = half-life corto → presupuestar mantenimiento activo, no asumir estabilidad.
- **La capa de visuales (WebGL/Rive/scroll) es de doble filo:** gana el pitch pero puede dañar los CWV que LHCI gatea, exige `prefers-reduced-motion` (si no, pasivo de accesibilidad) y parte del B2B lo lee como forma-sobre-fondo. Regla: UNA, cuando el brief lo justifique; nunca default. Refuerza el "3 motores por trabajo" de motion-system (#5).

Decisión abierta (choca con regla vigente §6):
- El dev recomienda probar `marketplace add`/install/triggering YA con la 1 skill existente (las `description` son la superficie más propensa a fallar; no reservar todo el riesgo de install al final). Esto choca con §6 ("Claude Code reservado estrictamente para el build final; no abrir antes ni para probar instalación"). PENDIENTE de decisión de Carlos: mantener §6, o abrir una excepción de smoke-test de install temprano. No se toca §6 hasta que decida.

Registrado pero NO adoptado:
- "Shippear un sitio real antes de autorar las 7" (el dev lo marca como sugerencia, no crítico). Tensión: las fundaciones (security-headers, perf-gates, tokens) deben existir para shippear bien → huevo-gallina; y vamos a mitad (1/7 + 1 en curso). Se mantiene la cadencia; opción anotada, no ejecutada.

Mantenido FUERA del RESIDENT (público), en notas privadas de Carlos (Plan de Vida / Argos):
- Tesis de rentabilidad, fulfillment-vs-demanda, reuse-compression como métrica, nicho/segmento objetivo, CMS como definición de mercado, postura de gama/ticket y el criterio de cierre del objetivo "modelo rentable". Es inteligencia comercial específica de Carlos; no pertenece a un repo público. (El README sí podría nombrar el nicho como posicionamiento de producto si Carlos lo decide — eso no es secreto.)

### 2026-06-17 — web-security-headers · turn 4 (Decisiones) — HECHO

F1–F8 cerrados sobre el research del turn 3. Veredicto y pins reescritos en §3. Decisiones lockeadas:

- **F1 — entrega CSP:** `<meta>` hash-based nativo de Astro en estático (default). NO se fuerza header real en Cloudflare — `@astrojs/cloudflare@13.7.0` no soporta `staticHeaders`. `frame-ancestors`/`report-to`/`sandbox`/report-only van en `public/_headers`. Review-gate: revisar `staticHeaders` en cada minor del adapter; si aterriza, reconsiderar header unificado.
- **F2 — headers no-CSP:** `public/_headers` canónico (todos los assets estáticos). Middleware (`src/middleware.ts`) SOLO para rutas on-demand/SSR (donde `_headers` no aplica).
- **F3 — nonce:** hash-only. Inline de tercero → hash en `scriptDirective.hashes`. Nonce real exige SSR + header; fuera de alcance del `<meta>` nativo.
- **F4 — COOP/COEP/CORP:** COOP `same-origin` en baseline; **COEP OFF por default** (`require-corp` rompe assets cross-origin de OGL #6 / Rive #7); CORP `same-origin` (aflojar a `cross-origin` para assets embebibles). Documentar cómo prender COEP solo si todo cross-origin manda CORP.
- **F5 — SRI:** no nativo. Para un sitio premium mayormente same-origin, SRI es OPCIONAL: documentar `integrity=` manual + `@kindspells/astro-shield@≥1.3.2` (1.7.1) como integración SOLO cross-origin y SOLO para SRI (su CSP solapa el nativo). Review-gate por CVE/mantenimiento.
- **F6 — cache:** el adapter v13 ya inyecta Cache-Control immutable para `_astro/*`. NO meter Cache-Control custom en `_headers` (quirk #13164, poco fiable). Caching de perf se difiere a `perf-ci-gates` (#3).
- **F7 — granularidad references:** 5 archivos — `csp-astro-native.md`, `cloudflare-headers.md` (incl. `.assetsignore`), `header-inventory.md` (tabla valor-por-header), `middleware-ssr.md`, `sri-and-verification.md`. `SKILL.md` carga veredicto + split `<meta>`/header + baseline `_headers` + config mínima.
- **F8 — Pages vs Workers Static Assets:** Workers Static Assets canónico (Pages removido en adapter v13); `_headers` porta a Pages.

Gobierno (decisiones de Carlos este turn):
- §6 SIN CAMBIOS: no se abre Claude Code para smoke-test temprano de install (resuelve el pendiente del entry de revisión externa — Carlos decidió No).
- Skill **`cms-self-edit`** (placeholder) planeada post-7: turno de selección de herramienta primero (gap de stack-canon — no hay CMS web), luego cadencia normal; scaffold al repo diferido. Registrada en §3 y §10.

Siguiente: web-security-headers turn 5 (Build — autoría del bundle → `quick_validate` → `package` → commit).

### 2026-06-17 — web-security-headers · turn 5 (Build) — HECHO

Bundle autoreado y commiteado (Git Data API, commit atómico de 6 archivos, sha 4f37a05): `skills/web-security-headers/SKILL.md` + 5 references.

- SKILL.md (120 líneas): veredicto + split meta/header-real + tabla de capas + receta de 5 pasos (security.csp mínimo, `public/_headers` baseline, middleware SSR, SRI opcional, verificación) + gotchas + limitations. Frontmatter solo name+description (validador); description 953 chars. Cuerpo en md-house-style — sin bold/itálicas/hr, solo #/##/###, "load when relevant".
- references: `csp-astro-native.md` (security.csp, runtime API, hashes externos, incompatibles, dev caveat), `cloudflare-headers.md` (_headers sintaxis/matching + `.assetsignore` + quirk #13164 + compat Pages), `header-inventory.md` (tabla valor-por-header + HSTS preload + trío cross-origin + XFO vs frame-ancestors + Managed Transforms), `middleware-ssr.md` (headers en rutas on-demand + CSP header real SSR + nonce), `sri-and-verification.md` (SRI manual/astro-shield ≥1.3.2 + verificación). Cada una con front matter ligero y una dirección de profundidad.
- Validación: reproducidas las reglas EXACTAS de `quick_validate.py` (un solo SKILL.md; frontmatter solo {name, description, license, allowed-tools, metadata, compatibility}; name kebab ≤64; description ≤1024 sin `<`/`>`) → PASS. El entorno de este chat es solo-Composio; el script canónico de skill-creator (`quick_validate` + `package_skill` reales) corre en el container local. La validación canónica + empaque oficial quedan para el BUILD FINAL de las 7 vía Claude Code (ya en plan, §6 y §9) — gate único, no per-skill.
- Empaque: `web-security-headers.skill` (zip del árbol, raíz `web-security-headers/`, 12.9 KB) subido como artefacto descargable para Carlos. El repo guarda las FUENTES, no el `.skill`.

Siguiente: perf-ci-gates turn 1 (Scoping).

### 2026-06-17 — perf-ci-gates · turn 1 (Scoping) — HECHO

Skill #3 (fundación). Tema: Lighthouse CI + Biome en GitHub Actions. Tool-selection ya lockeado por stack-canon (§2) → la skill codifica el CÓMO, no el QUÉ; NO requiere turno de selección (a diferencia de `cms-self-edit`). Sin discrepancia seed/§2/§3 este turno.

Reframe: la skill es un **gate de CI de dos puertas independientes** — (a) LHCI (`@lhci/cli`) gatea CWV/peso/categorías; (b) Biome gatea lint+format+import-order. Ambas en un `ci.yml`, jobs paralelos.

Orientación verificada (búsqueda ligera, re-verificar pins en turn 3):
- `@lhci/cli` 0.15.x trae Lighthouse 12.6.1; ~2M dl/mo; mantenido (abr-2026). PWA fuera desde LH12. LH13 exige Node 22.19+ y aún NO está en LHCI → watch de versión de Node.
- `treosh/lighthouse-ci-action` = action canónica (budget.json, numberOfRuns, `temporary-public-storage` default público 7 días). Es 3rd-party → SECONDARY vs `lhci autorun` crudo.
- Biome v2.4 (feb-2026); `biome ci` = lint+format+import-sort en un comando.
- **GOTCHA load-bearing — Biome × .astro es EXPERIMENTAL** (2.3+ landed, 2.4 mejoró): parse/lint/format de HTML/`<style>` exige `experimentalFullHtmlSupportEnabled:true`; reglas de lint cross-embedded "not supported yet". Decide F5.

Alcance — IN: config LHCI (`lighthouserc.json` + `budget.json`, assertions, target de colección, upload), config Biome (`biome.json`, `biome ci`, manejo .astro), el workflow `ci.yml`, targets de budget CWV 2026 + caveats de proxy de lab, control de flake (mediana-de-N, tiers warn/error), cómo servir un build Astro+Cloudflare para colección, wiring de status-checks. PARCIAL: `@lhci/server` self-host (solo la costura); run post-deploy contra preview URL (nombrado, no default). OUT (pointer): caching/`Cache-Control` runtime más allá del adapter (punt entrante de web-security-headers F6 → ver F8), SEO/schema profundo (→#4), a11y profundo + `prefers-reduced-motion` (→#5/#6/#7), RUM/field (CF Web Analytics, runtime no CI).

Forks abiertos (resolver turn 4) con sesgo:
- F1 target de colección LHCI: `staticDistDir:./dist` (sitio estático premium) · server-command (wrangler/preview) para rutas SSR · preview-URL post-deploy opcional.
- F2 estrategia de assertions: capas — category-score floors + budgets de métrica CWV + `budget.json` resource budgets. Lab NO mide INP → gatear **TBT** como proxy.
- F3 target de upload: `temporary-public-storage` default (cero infra, encaja en skill pública) · `@lhci/server` self-host en PikaPods como upgrade de historia durable.
- F4 runner LHCI: `npx @lhci/cli autorun` crudo (native-first, sin capa 3rd-party) vs `treosh` action (ergonómica, PR comments). Sesgo: crudo, honra "layers are a cost"; reconsiderar turn 4.
- F5 Biome en .astro: activar `experimentalFullHtmlSupportEnabled` (template completo) CON review-gate vs acotar Biome a script/TS/CSS/JSON + `prettier-plugin-astro` para template. El research decide.
- F6 invocación Biome en CI: `npx @biomejs/biome ci` vía script pnpm + `--reporter=github` (sin setup action extra).
- F7 forma del workflow: un `ci.yml`, dos jobs paralelos — `quality` (Biome, barato) ‖ `lighthouse` (build→serve→assert).
- F8 frontera cache/runtime: gate-only. Nota de una pantalla "qué perf asume el gate" (adapter hace `_astro/*` immutable; preload/preconnect; estrategia de imágenes). NO se vuelve workstream de config runtime — satisface el punt F6 mínimamente.
- F9 overlap de gating a11y/SEO con #4: asertar las 4 category floors de LH aquí (mismo run barato); SEO profundo→#4, a11y profundo→skills de visuales. Coarse-gate-aquí / deep-impl-en-otra.
- F10 granularidad references: ~4 — `lighthouse-config.md`, `github-actions-workflow.md`, `biome-setup.md`, `budgets-and-thresholds.md`. SKILL.md = veredicto + split de dos puertas + receta mínima.

Estructura del bundle propuesta: `SKILL.md` + references/{lighthouse-config, github-actions-workflow, biome-setup, budgets-and-thresholds}.

CASO CONTRARIO a defender en research:
- Primario: LHCI es lab-sintético y flakea en runners compartidos; el stack ya trae CF Web Analytics (RUM) con CWV de campo. Un gate pre-merge persigue un número que no matchea el campo y agrega fricción → solo Biome + RUM. La skill debe justificar LHCI como **gate de regresión pre-merge** (RUM es post-hoc, no bloquea un merge malo) y probar que se hace no-flaky (mediana-de-N, budgets en audits estables + tamaños de recurso, tiers warn/error, TBT-no-INP en lab). Si el research no lo defiende → veredicto pivota a "Biome gate duro + CF RUM; LHCI como run programado/manual opcional, no merge blocker".
- Secundario (tooling): soporte .astro de Biome aún experimental; ESLint+Prettier lintea mejor el template hoy. Research confirma si Biome 2.4 basta como herramienta única o si lo honesto mantiene `prettier-plugin-astro` para format de template (parte F5).

Checklist de research (turns 2→3): 1) re-verificar pins (`@lhci/cli`/LH/Node floor; Biome ≥2.4; `treosh`). 2) servir build Astro+`@astrojs/cloudflare@13` para LHCI — ¿`astro build` emite `dist/` estático→`staticDistDir`, o `_worker.js`? wrangler dev vs astro preview (F1). 3) shape de assertions LHCI 2026 + confirmar proxies de lab (TBT↔INP) (F2). 4) thresholds CWV 2026 (LCP ≤2.5s, INP ≤200ms, CLS ≤0.1) + budgets de lab sensatos + mitigación de flake (numberOfRuns, throttling). 5) Biome .astro en 2.4 — ¿qué tan bueno `experimentalFullHtmlSupportEnabled`? cobertura de lint del template? ¿aún `prettier-plugin-astro`? (F5). 6) comportamiento exacto `biome ci` + GitHub/multi-reporter (2.4) (F6). 7) `temporary-public-storage` vs `@lhci/server` self-host en PikaPods (F3). 8) `lhci autorun` crudo vs `treosh` action — delta de features, story de PR-comments, peso native-first (F4). 9) qué cachea `@astrojs/cloudflare@13` auto vs qué queda (acota F8).

Decisión de Carlos pendiente (1): confirmar frontera F8 — `perf-ci-gates` se queda como gate de CI y el punt de caching entrante se satisface con nota "perf asumida", NO workstream de caching runtime (sesgo fuerte; caching runtime es homeless en las 7, probablemente pertenece al playbook diferido).

Sandbox suffix activo este chat: `pcg_9jv35m`.

Siguiente: perf-ci-gates turn 2 (Pre-research).

### 2026-06-17 — perf-ci-gates · turn 2 (Pre-research) — HECHO

Skill `pre-research` corrida (4 fases). Phase 1 = proceder con supuestos (sin preguntas; F8 se lleva como supuesto declarado — gate-only, nota "perf asumida", no workstream de caching). Phase 2 = pre-investigación substantiva (8 búsquedas entre turns 1–2 + fuentes primarias). Pre-brief de 8 subtasks entregado en chat para Research mode (turn 3). TODO LO DE ABAJO ES PRE-INVESTIGACIÓN — re-verificar en turn 3.

Hallazgos clave (pre-investigación):
- **LHCI vivo y canónico, lab-only por diseño:** `@lhci/cli` 0.15.x trae Lighthouse 12.6.1; ~2M dl/mo; mantenido (abr-2026). PWA fuera desde LH12. LH13 exige Node 22.19+ y NO está aún en LHCI; 0.15.x corre en Node 18+. Runners Ubuntu traen Chrome en `/usr/bin/google-chrome`. **INP es field-only — lab usa TBT como proxy.** FCP/TBT solo-lab; LCP/INP/CLS pueden estar en ambos pero solo el percentil de campo cuenta para CWV. → Responde el caso contrario: LHCI = gate de regresión pre-merge sobre señales de lab estables (TBT, CLS, resource budgets) + proxies (LCP/FCP); CF RUM = verdad de campo post-deploy. "Passing beats perfection": no perseguir score perfecto.
- **Serving path Astro 6 + CF más rico que en turn 1:** `astro preview` ahora corre en **workerd** (paridad de producción, v13) y `astro dev`/`preview` usan el Cloudflare Vite plugin (workerd, no Node). Con worker presente, los assets estáticos viven en `dist/client/*` (no `dist/`). → Fork de colección LHCI: `astro preview` (paridad workerd, captura headers immutable del adapter) vs `staticDistDir: ./dist/client` (determinista, sin worker). Trampa que favorece el server path: build con base URL absoluta + LHCI sirviendo en puerto random de localhost → assets 404.
- **Biome `.astro` real pero experimental opt-in:** v2.3 metió soporte full Vue/Svelte/Astro (lint/format de JS/TS en script + CSS en style + template por reglas HTML de Biome), gated en `html.experimentalFullSupportEnabled`. Flag OFF = solo extrae JS/TS, ignora el resto. v2.4 mejoró parsing + bajó falsos positivos (noUnusedVariables/useConst/useImportType/noUnusedImports) SOLO con el flag ON. Caveat honesto (hilo contrario ESLint+Prettier): reglas cross-embedded aún no soportadas, formatting puede no matchear; roadmap 2026 lista "HTML stable + matchear Prettier + reglas HTML-ish" como pendientes. `biome ci` = formatter+linter+import-sort en un pase. OJO build: la key canónica es `html.experimentalFullSupportEnabled`; un blog v2.4 escribe `experimentalFullHtmlSupportEnabled` — verificar spelling en turn 3.
- **El runner casi no afecta el artefacto de config:** ambos consumen el mismo `lighthouserc.json` + `budget.json`. `lhci autorun` crudo pone PR status vía LHCI GitHub App (status a nivel PR/commit, link "Details", SIN artifacts/anotaciones GH-Actions). `treosh/lighthouse-ci-action@v12` (con el equipo Lighthouse + Treo) guarda resultados como action artifacts, soporta budgets que fallan el build, runs paralelos, `serverBaseUrl`/`serverToken` para server privado. Gotchas duros: `fetch-depth: 20`+ (shallow clone rompe detección de ancestro → "Could not find hash"); checkout del head sha en PRs. Storage default skill pública = `temporary-public-storage` (público, borra a 7 días); upgrade durable = `@lhci/server` self-host (Docker → PikaPods, encaja sesgo stack-canon).
- **CONFLICTO de fuentes a resolver:** mayoría 2026 mantiene LCP "good" <2.5s (LCP <2.5s, INP <200ms, CLS <0.1 @ p75). Un blog SEO (abr-2026) afirma que Google bajó LCP a 2.0s + subió INP a señal primaria (Search Central 18-mar-2026). Minoría vs varias fuentes → Research reconcilia contra primaria web.dev/Search Central, no resolver en silencio.

Subtasks de research (8, cap del skill) — versión completa en el pre-brief del chat:
1. Serving path (F1): `astro preview` workerd headless en CI + headers immutable, vs `staticDistDir ./dist/client`; caveat base-absoluta→404.
2. Shape de assertions LHCI + proxies (F2): `lighthouserc.json` 2026 (preset vs category minScore vs `maxNumericValue` FCP/LCP/TBT/CLS) + `budget.json` resource-summary; INP ausente en lab (TBT); mobile/desktop + límite de un solo status por run.
3. Thresholds CWV — reconciliación primaria (F4-threshold): LCP 2.0 vs 2.5 + reclasif INP contra web.dev/Search Central; traducir a budgets de LAB (más laxos, orientados a regresión).
4. Biome `.astro` en v2.4 (F5): confirmar key del flag, qué lintea/formatea ON vs OFF, fidelidad de format de template vs Prettier, `biome.json` recomendado (+ reglas a apagar si OFF).
5. `biome ci` + GitHub reporter (F6): semántica + exit codes, `--reporter=github` + multi-reporter 2.4, script pnpm.
6. Pins + Node floor (#1): `@lhci/cli`/LH/Node, Biome ≥2.4, `treosh@v12`, Astro 6/adapter 13.
7. Storage + mecánica de PR-status (F3): tokens autorun (LHCI GitHub App + PAT `repo:status`) vs treosh artifacts; esfuerzo de self-host `@lhci/server` (Docker→PikaPods).
8. Frontera de caching (F8): qué cachea `@astrojs/cloudflare@13` auto (`_astro/*` immutable confirmado) vs qué queda al autor → acota la nota de una pantalla.

(F7 forma del workflow, F9 overlap de category-floor con #4, F10 granularidad de refs = decisiones de turn 4, NO subtasks de research.)

Source priority: 1) docs primarias (docs.astro.build adapter CF, biomejs.dev config/language-support/v2.4, GoogleChrome/lighthouse-ci docs, web.dev/Search Central CWV); 2) repos/changelogs fuente (`withastro/astro`, `@astrojs/cloudflare`, `biomejs/biome`, `treosh/lighthouse-ci-action`); 3) Context7 (`@lhci/cli`, `biome`, `@astrojs/cloudflare`); 4) npm pins; 5) [SECONDARY] unlighthouse.dev (mismo autor, verificar), DEV/Medium, blogs SEO (el claim LCP-2.0 es uno de estos → lead, no hecho).

Estado del pre-brief: entregado con línea de confirmación. Carlos decide Continuar / Refinar / Abortar antes de turn 3 (toggle de Research mode es acción suya en la UI).

Sandbox suffix activo este chat: `pcg_9jv35m`.

Siguiente: perf-ci-gates turn 3 (Research — Research mode ON + Context7; re-verificar pins `@lhci/cli`/LH/Node, Biome ≥2.4, `astro@6.x`, `@astrojs/cloudflare@13.x`).

### 2026-06-17 — perf-ci-gates · turns 3–4 (Research + Decisions) — HECHOS

Research mode (turn 3) entregó reporte técnico verificado contra fuentes primarias (docs.astro.build, biomejs.dev, GoogleChrome/lighthouse-ci, treosh README, web.dev/Search Central, npm). Turn 4 lockea forks F1–F10.

Forks LOCKEADOS:
- F1 colección: `staticDistDir: ./dist/client` (sitio mostly-static premium; el adapter CF default es `output:'server'` → assets en `dist/client`, no `dist`). `startServerCommand`/`astro preview` (corre en workerd en Astro 6) SOLO si hay rutas SSR que auditar. Trampa #16276: con `base`/`site` absoluto los assets quedan en `dist/client/*` sin prefijo → 404 al servir literal; evitar `base` en el build auditado.
- F2 assertions: capas — preset `lighthouse:no-pwa` (PWA fuera desde LH12, no-pwa más seguro que recommended) + floors de categoría (performance error 0.9, accessibility error 0.95) + `maxNumericValue` por métrica (LCP/TBT/CLS error; FCP/SI/TTI warn) + `budget.json` resource budgets. OJO unidades: budget.json en KB, assertions LHCI en bytes/ms. INP NO medible en lab → TBT proxy (verificado web.dev).
- F3 upload: `temporary-public-storage` (público, borra a 7 días) + `uploadArtifacts`. Self-host `@lhci/server` (Docker → PikaPods; build token write-only seguro en repos públicos) = upgrade documentado para historia durable.
- F4 runner: **`treosh/lighthouse-ci-action@v12`** (REVERSAL del lean turn-1 "raw autorun por native-first"). Razón: raw `autorun` necesita la LHCI GitHub App instalada para PR status (MÁS integración externa, no menos); treosh no necesita app, guarda artifacts de Actions + public storage nativo, y consume el MISMO `lighthouserc.json` portable → cero lock-in del config (swappable a raw autorun después). Una action de marketplace es uso idiomático de Actions, no una capa de orquestación extra.
- F5 Biome `.astro`: flag **`html.experimentalFullSupportEnabled` OFF** + `prettier-plugin-astro` para format del template + 4 overrides cuando flag OFF (`style.useConst`, `style.useImportType`, `correctness.noUnusedVariables`, `correctness.noUnusedImports` → off para `.astro`/`.svelte`/`.vue`). (REVERSAL del lean turn-1/2 "flag ON con review-gate"). Razón: el formatter HTML de Biome sigue experimental, mete diffs breaking, y NO matchea Prettier (roadmap 2026 de Biome). Determinismo > bleeding-edge para una skill que codifica el stack. Split de ownership: Biome dueño de JS/TS/CSS/JSON; prettier-plugin-astro dueño del format de template `.astro`. EXCEPCIÓN stack-canon (dos formatters) — narrow, documentada; flipea a Biome-only cuando Biome marque HTML estable (review-gate). Carlos puede redirigir a flag-ON si prefiere comerse lo experimental. Key correcta = `html.experimentalFullSupportEnabled` (el blog v2.4 escribe `experimentalFullHtmlSupportEnabled` — INCORRECTO).
- F6 Biome CI: `npx @biomejs/biome ci --reporter=github` (el reporter debe ser explícito — auto-detección NO implementada, bug #9109). Biome pineado `--save-exact` como devDep. `biome ci` = formatter+linter+import-sort en un pase, sin `--write`, exit ≠0 en error. v2.4+ multi-reporter + `--reporter-file` opcional.
- F7 workflow: un `ci.yml`, 2 jobs paralelos — `quality` (Biome, barato) ‖ `lighthouse` (build→collect→assert→upload), ambos required checks para merge. Node 22, `npm ci`, `fetch-depth: 20` (shallow rompe detección de ancestro LHCI → "Could not find hash"), checkout del PR head SHA, `numberOfRuns: 5` (mediana de 5 = 2x más estable que 1 run; Chrome preinstalado en runners Ubuntu en `/usr/bin/google-chrome`).
- F8 caching: GATE-ONLY (lean turn-1 confirmado, Carlos no objetó). Nota de una pantalla "perf asumida": adapter auto-inyecta `Cache-Control` immutable (~1 año) en assets hasheados `_astro/*`; assets NO-hasheados vía `public/_headers` pero limitado por quirk de Workers Static Assets (#13164, wontfix/has-workaround: `_headers` Cache-Control no honrado para assets estáticos, vuelve `max-age=0, must-revalidate`). Watch: #16692 (hashed assets no cacheados en algunas configs adapter 13.5.0). NO workstream de caching runtime.
- F9 overlap con #4: floors coarse aquí — performance+accessibility `error`, best-practices+seo `warn` (deja el gate duro de SEO a #4). Deep SEO/schema → #4; deep a11y + prefers-reduced-motion → skills de visuales (#5/#6/#7).
- F10 refs: 4 — `lighthouse-config.md` (lighthouserc + budget.json + serving path), `github-actions-workflow.md` (ci.yml completo, ambos jobs, caching, fetch-depth, status checks), `biome-setup.md` (biome.json + flag .astro + biome ci), `budgets-and-thresholds.md` (CWV 2026, proxies de lab TBT↔INP, control de flake). SKILL.md = veredicto + split de dos puertas + receta mínima + gotchas + limits.

Caso contrario DERROTADO:
- Primario (LHCI flaky/redundante con CF RUM): LHCI = gate de regresión PRE-MERGE (mediana de 5, budgets de lab orientados a regresión, TBT como proxy de INP); CF RUM = verdad de CAMPO post-deploy. Complementarios, no redundantes. "Passing beats perfection" — no perseguir score perfecto.
- Conflicto de fuentes RESUELTO: claim "LCP a 2.0s / INP reclasificado a primario / post Search Central 18-mar-2026" = FALSO. Ninguna fuente primaria de Google lo confirma; web.dev y Search Central (últ. act. 2025-12-10) mantienen LCP 2.5s / INP 200ms / CLS 0.1 @ p75. INP es CWV completo desde el swap FID→INP del 12-mar-2024. digitalapplied.com (citada por los blogs SEO) ella misma desmiente el 2.0s. El "alertar a 80% del threshold (LCP 2.0/INP 160)" es un buffer auto-impuesto, no un threshold de Google — origen probable de la confusión. Veredicto rechaza 2.0s explícitamente.

Pins verificados 2026-06-17 (npm/repos, con fechas): `astro@6.4.7` (Node 22+; watch 7.0.0-alpha.2/beta.3); `@astrojs/cloudflare@13.7.0` (solo Workers, no Pages; peer wrangler@^4.83); `@lhci/cli@0.15.1` (LH 12.6.1; piso histórico Node ≥18 pero Astro 6 fuerza 22; LH13 exige Node 22.19+ y NO está aún en LHCI); `@biomejs/biome@2.5.0` (508 reglas, cross-file lint; conservador 2.4.16 del 2026-05-27); `treosh/lighthouse-ci-action@v12` (tag vivo → 12.6.2, node24, LH 12.6).

Confianza: pins/thresholds/key-de-flag/quirk #13164/mecánica de tokens (`LHCI_GITHUB_TOKEN` PAT `repo:status` vs `LHCI_GITHUB_APP_TOKEN`)/gotchas de CI = VERIFICADO (fuentes primarias). Números de budget de lab = INFERIDO (calibrar al valor medido real del sitio, ~110-115% del actual, luego ratchet down).

Artefacto de research (reporte completo) generado en el chat de este turno — reusar en turn 5 para los 4 refs.

Sandbox suffix activo este chat: `pcg_9jv35m`.

Siguiente: perf-ci-gates turn 5 (Build) — autoría de SKILL.md + 4 refs, `quick_validate` (description ≤1024, sin `<`/`>`, sin `: ` mid-string; frontmatter solo {name, description, license, allowed-tools, metadata, compatibility}), package `.skill`, commit del fuente. Re-verificar pins al inicio del build.

### 2026-06-17 — perf-ci-gates · turn 5 (Build) — HECHO · skill #3 COMPLETA

NOTA DE RECUPERACIÓN: turn 5 se reanudó tras un corte a mitad del commit. El run cortado YA había autorado los 5 archivos (vía code-exec) y empacado el `.skill`; esos archivos persistieron en el contenedor de code-exec (entorno distinto al sandbox de Composio). En la reanudación se confirmó por API que el commit de fuente NO había aterrizado (HEAD seguía en turn-4 `cf93847`, y `skills/perf-ci-gates/` solo tenía el stub de 1459 B sin references). Se reusaron los archivos persistidos tras verificarlos uno por uno contra los forks F1–F10 lockeados, se re-validó y se commiteó. Cero reautoría a ciegas.

Bundle: `skills/perf-ci-gates/SKILL.md` (13.4 KB, 215 líneas) + `references/` (lighthouse-config.md, github-actions-workflow.md, biome-setup.md, budgets-and-thresholds.md). Convenciones de casa seguidas (frontmatter name+description solo; body `#/##/###`, sin bold/italics/HR; tablas; TOC en los 3 refs >100 líneas; budgets-and-thresholds 55 líneas sin TOC). Refleja los forks lockeados exactamente: F1 `staticDistDir ./dist/client` (+ astro preview SSR, trampa base absoluta); F2 assertions en capas, preset `no-pwa`, floors perf/a11y error + best-practices/seo warn, métricas LCP/TBT/CLS error + FCP/SI/TTI warn, `budget.json` (KB vs bytes); F3 `temporary-public-storage` + uploadArtifacts, self-host `@lhci/server` documentado; F4 `treosh/lighthouse-ci-action@v12` (sin lock-in, variante raw autorun + mecánica de tokens en ref); F5 flag `.astro` OFF + `prettier-plugin-astro` + formatter Biome OFF para `.astro` + 4 overrides; F6 `biome ci --reporter=github` explícito; F7 un `ci.yml`, 2 jobs, fetch-depth 20, PR head SHA, numberOfRuns 5; F8 nota gate-only (#13164); F9 floors coarse. CWV sin cambio (2.5/200/0.1), claim 2.0s rechazado, INP→TBT.

Validación: `quick_validate` → "Skill is valid!". description 998/1024, frontmatter keys = {name, description}, sin `<`/`>`, sin `: ` mid-string. Lint house-style propio = 0 issues reales (2 falsos positivos por asteriscos de globs `**/*.astro` dentro de inline-code). `package_skill.py` → `.skill` 15319 B (15.3 KB), validó adentro, árbol completo (SKILL.md + 4 refs).

Commit de fuente: atómico vía Git Data API (5 blobs → tree sobre base `238ede14` → commit → update ref). Commit `23d8388f`, autor Carlos A. Cedillo L. / carlos@venturedge.com.mx. main → `23d8388f`. Verificado por API: SKILL.md (sha 3e8f9c, stub sobrescrito) + 4 refs presentes en `references/`.

Estado del repo: 3/7 skills redactadas y commiteadas (#1 astro-css-tokens, #2 web-security-headers 4f37a05, #3 perf-ci-gates 23d8388f). El `.skill` empacado se entrega al usuario en el chat (present_files). Recordatorio de gobernanza: Claude Code sigue reservado para el BUILD FINAL de las 7 juntas — no se abrió.

Sandbox suffix activo este chat: `pcg_9jv35m`.

Siguiente: `seo-aeo-schema` (#4, fundación) turn 1 (Scoping). Tema sembrado: `schema-dts` `@graph` tipado (`@id` cruzados) + `@astrojs/sitemap` + `llms.txt`. Pins sembrados a re-verificar: `schema-dts`, `@astrojs/sitemap`.

### 2026-06-17 — seo-aeo-schema · turn 1 (Scoping) — HECHO

Skill #4 (fundación). Tema sembrado: `schema-dts` `@graph` tipado (cross-`@id`) + `@astrojs/sitemap` + `llms.txt`. Tool-selection NO requerido (el stack-canon ya cubre el QUÉ; la skill codifica el CÓMO). Sin discrepancia seed/§3 este turno.

Reframe: la skill es la **capa de descubribilidad machine-readable** (qué es la página y cómo se relaciona con el resto del sitio, sin tocar un pixel). De los 3 pilares, solo schema.org JSON-LD tiene payoff probado hoy; sitemap = plumbing table-stakes; `llms.txt` = apuesta a un estándar no ratificado. El esfuerzo se pondera así.

Alcance — IN: `schema-dts` `@graph` centralizado por página (cross-`@id`: Organization↔WebSite↔WebPage↔Breadcrumb↔entidad) + render util tipado; `@astrojs/sitemap` opinionado; `robots.txt` generado (ref al sitemap + directivas AI-crawler); `llms.txt` (provisional — ver caso contrario); canonical + componente meta/OG/Twitter typed mínimo (F1, decisión de Carlos); receta de validación JSON-LD. PARCIAL (seam): i18n/hreflang; OG image dinámico (satori, constraint Workers). OUT (pointer): RSS (`@astrojs/rss`, content concern); catálogo exhaustivo de tipos schema; el GATE de SEO (se queda en #3 — F9 puso seo a `warn`; #4 autorea+valida, #3 gatea el merge).

Forks abiertos (resolver turn 4) con lean:
- F1 alcance meta/head: incluir componente `<Head>` typed mínimo (title/desc/canonical/OG/Twitter, hand-rolled, sin `astro-seo`) vs punt. **Lean: incluir** — DECISIÓN DE CARLOS pendiente (scope-expansion).
- F2 forma de emisión JSON-LD: `@graph` centralizado cross-`@id` vs bloques dispersos. Lean: centralizado.
- F3 herramienta de autoría schema: `schema-dts` + render util vs wrapper (`astro-seo-schema`) vs JSON a mano. Lean: schema-dts + util (sin wrapper); gated por caso contrario.
- F4 profundidad sitemap config: defaults vs opinionado (filter, serialize/lastmod, i18n seam). Lean: opinionado.
- F5 disposición `llms.txt`: shippear build-generated por default vs documentar-opcional vs omitir. Lean provisional: shippear flagged low-consensus; depende de research #5.
- F6 robots.txt + política AI-crawler: incluir (generado, ref sitemap, directivas UA) vs punt. Lean: incluir mínimo — robots.txt es el home *real* y aplicable del control de AI-crawlers (a diferencia de llms.txt).
- F7 validación schema en build: receta + check opcional (invocable por #3) vs solo receta. Lean: receta + check; el merge-gate se queda en #3.
- F8 OG image + RSS: out→pointer vs in. Lean: out→pointer.
- F9 i18n/hreflang: solo seam vs desarrollar. Lean: solo seam.
- F10 granularidad refs: ~4 — `meta-and-canonical.md`, `json-ld-graph.md`, `sitemap-config.md`, `robots-and-llms.md` (posible 5º catálogo de tipos en build).

CASO CONTRARIO (doble) a romper/defender en research:
- Primario — schema-dts es ceremonia sin valor runtime (solo tipos TS; los `@id` cross-ref se tipan como strings planos, sin chequeo referencial real). Un sitio con ~5 tipos podría autorar un JSON a mano y soltar la dep. La skill debe justificar schema-dts por seguridad de typo/refactor a escala de `@graph`, autocomplete sobre el vocabulario, mantenibilidad — o el veredicto pivota a JSON-LD a mano con shape tipado. Si schema-dts está stale/sin mantener → pivote forzado.
- Secundario — `llms.txt` es apuesta a estándar no adoptado (a jun-2026, ningún AI-crawler mayor ha confirmado honrarlo — INFERIDO, verificar). Shippear por default puede ser cargo-cult. La skill debe justificarlo como insurance barato de posicionamiento presente, o degradarlo a opcional y liderar AEO con lo que sí funciona (HTML semántico + JSON-LD + meta + sitemap + control AI-crawler en robots.txt).

Seams cross-skill (no perder):
- **JSON-LD × CSP (#2) — LOAD-BEARING.** JSON-LD se shippea inline (`<script type="application/ld+json">`); `script-src` aplica a esos bloques por spec CSP aunque no ejecuten. Si el CSP hash-based de Astro 6 NO hashea bloques `ld+json`, un `script-src` estricto de #2 puede BLOQUEAR la structured data en silencio. Tope del research.
- SEO floor (#3): #3 puso `best-practices`/`seo` a `warn` (coarse); #4 autorea+valida (deep). Componen: coarse-gate en #3, deep aquí.

Checklist de research (turns 2→3): 1) pins (`schema-dts` last-publish/mantenimiento; `@astrojs/sitemap` vs astro@6.4.7/Node22). 2) typing `@graph` de schema-dts (`WithContext`/`Graph`, si los `@id` cross-ref se tipan o son strings, render helper, gotchas de profundidad/union, `@context`). 3) `@astrojs/sitemap` bajo astro@6 + cloudflare@13 (opciones, descubrimiento de páginas, static-vs-SSR — adapter default `output:'server'`, ¿qué rutas entran?, output path `dist/client`?, entryLimit/index, i18n, lastmod). 4) **CSP×JSON-LD (×#2)**: ¿Astro 6 hashea `ld+json` inline? ¿`script-src` los gatea? failure mode + fix. 5) `llms.txt` 2026 (estado del spec + adopción real por AI-crawlers mayores — verificar; `llms.txt` vs `llms-full.txt`; generación static vs endpoint Astro desde content collections). 6) robots.txt + control AI-crawler (tokens UA exactos: GPTBot, ClaudeBot, Google-Extended, PerplexityBot, CCBot — verificar; generado vs static; ref sitemap; ¿robots.txt es el surface real vs llms.txt?). 7) payoff schema.org 2026 (tipos de rich-result vivos — verificar deprecación FAQ/HowTo; Article/Product/Breadcrumb/Organization/WebSite+SearchAction; evidencia de que AI Overviews/Perplexity/ChatGPT search consumen JSON-LD). 8) validación (Rich Results Test + Schema Markup Validator actuales; validación de JSON-LD en build invocable por #3). 9) patrón de static-endpoint Astro bajo cloudflare@13 para emitir text files (robots.txt, llms.txt): `prerender = true`, `GET → Response(text)`, dónde aterriza, vs drop estático en `public/`.

Estructura del bundle propuesta: `SKILL.md` + references/{meta-and-canonical, json-ld-graph, sitemap-config, robots-and-llms} (4; posible 5º).

Decisión de Carlos pendiente (1): F1 — ¿expandir alcance a componente meta/head typed mínimo (lean) o mantener #4 estricto a schema+sitemap+llms?

Sandbox suffix activo este chat: `sas_k4m2q9`.

Siguiente: seo-aeo-schema turn 2 (Pre-research).

### 2026-06-17 — seo-aeo-schema · turn 2 (Pre-research) — HECHO

Skill `pre-research` corrida (4 fases). Phase 1 = proceder con supuestos (sin preguntas; F1 resuelto, territorio claro). **F1 RESUELTO → IN** (decisión de Claude, delegada por Carlos): la skill incluye componente meta/head typed mínimo (title/desc/canonical/OG/Twitter, hand-rolled, sin `astro-seo`). Phase 2 = pre-investigación substantiva (5 búsquedas + 1 fetch + fuentes primarias). Pre-brief de 8 subtasks + source priority entregado en chat. TODO LO DE ABAJO ES PRE-INVESTIGACIÓN — re-verificar en turn 3.

Hallazgos clave (pre-investigación, con citas en el chat del turn):
- **schema-dts NO está stale — lo contrario.** Major fresco 2.0.0 (23-mar-2026, npm), mantenimiento healthy, ~0.95–1.16M dl/semana. Expone `WithContext<T>`, interfaz `Graph` (estructuras interconectadas con refs `@id`), `Role<T,P>`. → DERROTA en gran parte el caso contrario primario ("schema-dts muerto, autorar a mano"). Resta: ¿vale el dep el surface de 2.0.0?, ¿los `@id` cross-ref se type-checkean o son strings planos?, pin 2.0.0 vs conservador 1.1.5.
- **@astrojs/sitemap 3.7.3** (~3 sem, core-team, actualizado a Astro 6). LIMITACIÓN DURA: NO genera entradas para rutas dinámicas en SSR mode ni analiza el source de la página. Como el adapter CF default es `output:'server'`, solo rutas prerendered entran al sitemap → rutas SSR necesitan `customPages`/`serialize` o endpoint custom. Output `sitemap-index.xml` + `sitemap-0.xml`. Opciones: `filter`, `serialize`, `entryLimit` (45000), `namespaces` (3.6.0+), changefreq/priority (IGNORADOS por Google). CAUTION: 3.7.1 tuvo regresión de build ('reduce' undefined, #15894) → verificar 3.7.3 limpio en astro@6.4.7.
- **Seam CSP×JSON-LD (×#2) — REAL y SIN RESOLVER.** Astro CSP hashea SUS scripts bundleados (islands); scripts externos NO soportados de fábrica (provees tu hash). JSON-LD inline autorado NO es "bundled script" → ¿Astro lo hashea? Abierto. Por spec, `script-src` gobierna scripts inline (allow vía unsafe-inline/nonce/hash que matchee) — PERO JSON-LD es data block que NO ejecuta; si los browsers aplican `script-src` a data blocks es EL crux. Compone con #2: si solo el `<meta>` lleva hashes, el script se permite solo donde header Y meta permiten; el `<meta>` solo puede restringir más. Trampa de determinismo: hashes de inline-scripts por-página varían en CI por chunk boundaries de Vite (pero un bloque ld+json estático tiene hash content-stable). → TOP del research turn 3.
- **llms.txt: contested; el framing honesto INVIERTE el instinto del seed.** Consenso (varios estudios may-2026): ningún proveedor LLM mayor ha comprometido públicamente honrar llms.txt como señal en producción; Google (Illyes, jul-2025) dijo NO explícitamente; interés de crawler negligible (408 de 500M+ eventos; ~10.13% adopción de dominios); Semrush sin correlación estadística. COMO SEO play = inútil. PERO el uso legítimo es otro: capa Business-to-Agent (routing para agentes); OpenAI/Anthropic/Stripe/Cloudflare/Vercel usan llms.txt como routing layer para agentes de coding; Chrome Lighthouse 13.3 (7-may-2026) ahora lo audita. → F5 resuelve hacia: SHIPPEAR, framed como posicionamiento B2A barato, NO como palanca SEO; liderar AEO con lo que sí funciona. NOTA: una fuente (whatsmygeoscore) afirma "plataformas comprometidas para contenido post-ene-2026" — choca con las fuentes fuertes y conflaciona llms.txt con bloqueo de crawlers; LOW-RELIABILITY, verificar contra docs primarios de vendor.
- **robots.txt = el surface REAL (aunque imperfecto) de control AI-crawler.** Bots reputables lo honran; scrapers agresivos lo ignoran. Tokens surfaced (verificar exactos turn 3): GPTBot + OAI-SearchBot + ChatGPT-User (OpenAI), ClaudeBot + Claude-SearchBot (Anthropic), Google-Extended (Google), PerplexityBot, CCBot (Common Crawl), Meta-ExternalAgent (Meta), Applebot-Extended (Apple), Bingbot (MS). NUANCE CRÍTICO: bloquear un crawler ≠ mantener la página fuera de una respuesta AI — para excluir del todo usar noindex/auth; no contradecir robots.txt vs llms.txt.
- **Payoff de schema 2026 más estrecho y actual de lo que asumía el seed.** Google deprecó FAQ rich results el 7-may-2026 (RRT pierde FAQ jun-2026, Search Console API ago-2026; FAQPage sigue válido como tipo y parseado; rankings NO afectados). HowTo ya muerto; 7 tipos retirados jun-2025 (Book Actions, Course Info, Claim Review, Estimated Salary, Learning Video, Special Announcement, Vehicle Listing). SIGUEN dando rich results: Product, Review/AggregateRating, Article, Recipe, Video, Organization, LocalBusiness, BreadcrumbList, Event. HECHO AEO-definitorio (guía de Google): NO se necesita schema especial para AI Overviews / AI Mode, aunque la structured data debe matchear el contenido visible. Organization/Person = esenciales para reconocimiento de entidad en sistemas AI. → Valor honesto del JSON-LD = reconocimiento de entidad + rich results sobrevivientes + parseo correcto, NO palanca mágica de ranking AI. Palancas AEO reales son content-side: citas/quotes/estadísticas suben visibilidad hasta 40% (Princeton/Georgia Tech); 86% de citas AI vienen de fuentes brand-controlled (Yext).

Subtasks de research (8, cap del skill — versión completa en el pre-brief del chat):
1. schema-dts 2.0.0: breaking changes vs 1.x, `WithContext`/`Graph`/`Role`, ¿`@id` type-checked o strings?, pin 2.0.0 vs 1.1.5.
2. **CSP×JSON-LD (×#2) LOAD-BEARING**: ¿`script-src` bloquea data blocks `ld+json`? ¿Astro 6 `security.csp` hashea ld+json inline autorado o solo sus bundled? fix si hace falta hash. → patrón ship-safe bajo el `<meta>` CSP de #2.
3. @astrojs/sitemap 3.7.3 en el stack: regresión 3.7.1 fija?, límite SSR + cómo incluir rutas SSR (serialize/customPages vs endpoint), output path (dist/client?), surface de opciones.
4. llms.txt: ¿algún vendor lo honra en prod (docs primarios)?, Lighthouse 13.3 audita?, framing B2A + `llms.txt` vs `llms-full.txt`, patrón de generación en Astro (public/ vs endpoint prerendered desde content collections).
5. robots.txt: tokens UA exactos de cada vendor (docs oficiales), surface enforceable + nuance disallow-≠-fuera-de-respuestas.
6. payoff schema 2026 + `@graph` canónico: tipos de rich-result vivos + estado deprecación (Search Central), guía AI-features "no special schema, match visible content", entity graph baseline (Organization↔WebSite[+SearchAction]↔WebPage↔BreadcrumbList↔entidad).
7. validación + factibilidad CI: estado de Rich Results Test (pierde FAQ jun-2026) vs Schema Markup Validator; ¿validador JSON-LD programático/CI invocable por `ci.yml` de #3?, o validación type-time + manual. Dónde vive (#3 vs #4).
8. patrón text-endpoint Astro (cloudflare@13) para robots.txt + llms.txt (`prerender = true`, `GET → Response(text)`), dónde aterriza, endpoint-from-collections vs drop static en public/.

Source priority (completa en chat): 1) docs primarios (docs.astro.build `security.csp`/sitemap/endpoints/build-with-ai, @astrojs/cloudflare; Google Search Central structured-data/AI-features/crawlers/rich-results; W3C CSP3 + MDN; schema.org + validator; spec llms.txt llmstxt.org). 2) docs de crawler de vendor (OpenAI/Anthropic/Google-Extended/Perplexity/Common Crawl). 3) repos/changelogs (`withastro/astro` sitemap CHANGELOG + CSP #14798/#6407 + roadmap #377; `google/schema-dts` 2.0.0 notes; `@astrojs/cloudflare`). 4) Context7 (@astrojs/sitemap, schema-dts, astro, @astrojs/cloudflare). 5) npm pins. 6) [SECONDARY] estudios GEO con metodología nombrada (Limy, SE Ranking, OtterlyAI, paper Princeton/Georgia Tech, Yext) solo para cifras de adopción; claims de blogs SEO "plataformas comprometidas con llms.txt" = leads low-reliability.

Estado del pre-brief: entregado con línea de confirmación (Continuar / Refinar / Abortar). Carlos decide antes de turn 3; el toggle de Research mode es acción suya en la UI.

Sandbox suffix activo este chat: `sas_k4m2q9`.

Siguiente: seo-aeo-schema turn 3 (Research — Research mode ON + Context7; re-verificar pins schema-dts 2.0.0 vs 1.1.5, @astrojs/sitemap 3.7.3, astro@6.4.7, @astrojs/cloudflare@13.7.0, Node 22).

### 2026-06-17 — seo-aeo-schema · turns 3–4 (Research + Decisions) — HECHOS

Research mode (turn 3) entregó reporte técnico verificado contra fuentes primarias (google/schema-dts releases, docs.astro.build, Google Search Central, W3C/MDN, llmstxt.org, docs de crawler de vendors, npm). Turn 4 lockea F2–F10 y reescribe §3 veredicto/pins. Reporte completo en el chat del turn 3 (reusar en turn 5 para los 5 refs).

Hallazgos VERIFICADOS (fuente primaria salvo nota):
- **Pins (npm/GitHub, jun-2026):** `schema-dts@2.0.0` (23-mar-2026, ~1.7M dl/sem, TS-only, sin runtime); `@astrojs/sitemap@3.7.3`; `astro@6.4.7`; `@astrojs/cloudflare@13.7.0`; Node 22.
- **schema-dts 2.0.0 — breaking vs 1.x** (release v2.0.0, commit a6aad64): Role no-recursivo (#205); Quantity ahora DataType core (Schema.org v30) → asignaciones antes legales pueden errar; soporta Schema.org v30; añade `WithActionConstraints<T>`, `MergeLeafTypes`, tipos `…Leaf`; schema-dts-gen ahora depende de nuevo paquete `schema-dts-lib` (#213). Tipos: `WithContext<T> = T & {"@context"}` (en 2.0.0 SOLO intersección — ya NO puede ser Graph; importar `Graph` aparte); `interface Graph { "@context"; "@graph": readonly Thing[] }`; `Role<T,P>`. Import: `import type { WithContext, Graph, Person } from 'schema-dts'`.
- **CRÍTICO — @id NO se type-checkea** (verificado contra source + README): `@id` es `IdReference = {"@id": string}`, string plano. Cero validación referencial — no verifica que el `@id` referenciado exista en el graph ni que apunte al tipo correcto. Props objeto se generan como `SchemaValue<T | IdReference>` → cualquier string pasa. NET: el dep compra seguridad de prop/tipo POR NODO; el wiring del graph es responsabilidad del autor + validación runtime. Gotcha de profundidad: uniones discriminadas en `@type` son grandes; multi-tipo via `MergeLeafTypes<[ProductLeaf, …Leaf]>`.
- **CSP × JSON-LD (×#2) — RESUELTO: NO-ISSUE.** (a) `<script type="application/ld+json">` es data block (HTML spec) que NO ejecuta → CSP `script-src` gobierna ejecución → browsers NO lo bloquean, NO dispara violación (W3C/WHATWG + MDN; confirmado por análisis canónico de Mathias Bynens). (b) Astro `security.csp` hashea solo scripts ejecutables que bundlea (islands, define:vars) — NO hashea ld+json autorado, y NO hace falta. (c) Fix requerido: NINGUNO para ld+json. En Cloudflare 13.7.0 (sin staticHeaders) el CSP va por `<meta>` — OK porque no hay hash involucrado. (d) El drift CI/local de hashes (chunk boundaries de Vite) NO afecta a un bloque ld+json estático (sin hash). PATRÓN SHIP-SAFE: `<script type="application/ld+json" set:html={escape(JSON.stringify(graph))} />` con escape de `<>&` (XSS/tag-breakout); habilitar `security.csp` para scripts ejecutables; NO tocar `scriptDirective.hashes` por el ld+json. → El riesgo #1 del skill queda desactivado.
- **@astrojs/sitemap 3.7.3:** regresión 3.7.1 #15894 ('reduce' undefined) fija; 3.7.3 builda en astro@6.4.7 (MEDIUM en el commit exacto). Límite duro (docs): NO analiza source de página → NO descubre rutas dinámicas/SSR (`output:'server'`). `customPages` espera array estático (función async → error). Incluir rutas SSR: (1) `customPages:[...]`; (2) endpoint `sitemap.xml` prerendered propio desde content collections; (3) mantener rutas prerendered. Opciones: `site` (en config raíz, NO en `sitemap()`), `filter`, `serialize`, `entryLimit` (45000), changefreq/lastmod/priority (changefreq+priority IGNORADOS por Google), `namespaces` (3.6.0: news/xhtml/image/video), `i18n`, `prefix`, `customPages`, + `xslURL`/`customSitemaps`/`chunks` (3.7.0). BUG #16838 (repro ≤3.7.2): `<lastmod>` no se escribe en entries de `sitemap-index.xml` aunque serialize lo ponga por-URL. lastmod via `serialize(item){ item.lastmod = new Date(date) }` desde fecha de collection/git mtime. Output: `dist/client` bajo el adapter server (consistente con #1–#3; MEDIUM, confirmar en build).
- **llms.txt — CONFLICTO resuelto contra primaria:** claims de blogs SEO ("plataformas comprometidas post-ene-2026") NO respaldados por docs de vendor. Google (Illyes, Search Central Deep Dive Bangkok jul-2025, vía Schwartz): Google NO crawlea ni soporta llms.txt, sin planes; Mueller ~ keywords meta tag; la guía AI de Google (may-2026) lo lista en mythbusting. Adopción ≠ consumo: SE Ranking (300k dominios) = 10.13% adopción; Ahrefs (137k dominios) = 97% nunca leídos, bots AI = 1% de requests; Limy (500M+ visitas bot/90d) = 408 a llms.txt (negligible). Lighthouse 13.3 (7-may-2026) SÍ añadió audit de llms.txt (categoría Agentic Browsing, default config; chequea presencia + H1 + links + no muy corto; "Not Applicable" en 404; ratio fraccional, NO score 0–100; agent-readiness, NO ranking). `llms.txt` = índice Markdown curado (H1 nombre, blockquote resumen, H2 de links, sección Optional); `llms-full.txt` = doc entera inlined. Uso legítimo B2A: agentes de coding / docs-sites (Cloudflare, Vercel, Anthropic, Mintlify auto-genera ambos; formato co-desarrollado Mintlify+Anthropic). Generación Astro: endpoint prerendered desde content collections (mejor) o `public/llms.txt` estático. VEREDICTO F5: SHIPPEAR como agent-readiness/B2A/Lighthouse, NO palanca SEO; framing honesto "no hace nada para superficies AI de Google".
- **robots.txt — tokens verificados (jun-2026):** OpenAI — GPTBot (training), OAI-SearchBot (índice ChatGPT search/citas), ChatGPT-User (fetch user). Anthropic (support.claude.com art. 8896518) — ClaudeBot (training), Claude-SearchBot (índice search), Claude-User (fetch user); legacy: Claude-Web, anthropic-ai. Google — Googlebot (search; debe ir allow para AI Overviews/AI Mode), Google-Extended (token robots, NO crawler; opt-out de training/grounding Gemini/Vertex; NO afecta ranking ni saca de AI Overviews). Perplexity — PerplexityBot (índice/citas), Perplexity-User (puede NO obedecer robots.txt). Common Crawl — CCBot. Meta — Meta-ExternalAgent (training/índice), FacebookBot; NO bloquear facebookexternalhit (previews). Apple — Applebot (search), Applebot-Extended (opt-out Apple Intelligence). MS — Bingbot. NUANCES: (1) robots.txt = surface enforceable (vs llms.txt que no es mecanismo de acceso). (2) bloquear crawler ≠ fuera de respuestas AI → usar noindex/auth/nosnippet. (3) Googlebot debe ir allow para que AI features de Google usen el contenido; Google-Extended solo gobierna training. (4) fetchers user-initiated inconsistentes. VEREDICTO F6: baseline allow search + AI-search; training opt-out = decisión documentada (comentada).
- **Schema 2026 payoff (Google Search Central):** rich results vivos — Product, Review/AggregateRating, Article, Recipe, Video, Organization, LocalBusiness, BreadcrumbList, Event. FAQ deprecado 7-may-2026 (RRT pierde FAQ jun-2026; SC API ago-2026; FAQPage sigue válido y parseado; rankings NO afectados). HowTo muerto. 7 tipos retirados jun-2025. AI-features (quotes directos): "no additional requirements to appear in AI Overviews or AI Mode", "no special schema.org structured data you need to add", "you don't need to create new machine readable files… or markup"; PERO la structured data debe matchear contenido visible. Organization/Person = clave para entity recognition. @graph canónico cross-@id: Organization(#org) ← WebSite(#website, publisher→#org, + SearchAction) ← WebPage(#webpage, isPartOf→#website, breadcrumb→#breadcrumb) + BreadcrumbList(#breadcrumb) + entidad por página (Article #article, isPartOf/mainEntityOfPage→#webpage, publisher→#org). QAPage (multi-respuesta de usuarios) vs FAQPage (Q&A del owner) — no confundir. FRAMING AEO: JSON-LD NO da inclusión directa en AI Overviews/AI Mode/citas; da elegibilidad de rich results + claridad de entidad; las superficies AI extraen del texto visible (que debe matchear). Palancas AEO reales = content-side (citas/quotes/stats +40% Princeton/GT; 86% citas de fuentes brand-controlled, Yext).
- **Validación:** Rich Results Test (solo tipos Google; pierde FAQ jun-2026) vs validator.schema.org (sintaxis/compliance, sin elegibilidad Google). Validadores programáticos: `sdtt`/structured-data-testing-tool (CLI+lib, presets, Jest/headless) PERO deps ~2019 — verificar mantenimiento; schema-dts = solo compile-time. VEREDICTO F7: codificar (1) safety compile-time schema-dts + (2) checks manuales documentados + EXPORTAR script custom (extrae ld+json del HTML built → JSON.parse → required props por tipo, sin dep stale) para que el `ci.yml` de #3 lo gatee. El workflow vive en #3, NO en #4.
- **Endpoint de texto (cloudflare@13.7.0):** por el `output:'server'` default, un endpoint de texto debe `export const prerender = true`. `export const GET: APIRoute = ({ site }) => new Response(body, { headers: { 'Content-Type': 'text/plain; charset=utf-8' } })`. robots.txt estático o llms.txt construido desde `getCollection()`. Prerendered → aterriza en `dist/client`, servido como asset estático (sin invocar Worker). Tradeoff vs `public/`: endpoint puede construir desde collections (build-time); `public/` es fijo. Caveats CF: `.assetsignore` en `public/`; `public/_headers` controla headers de assets (sin staticHeaders en 13.7.0); cuidar que routing/.assetsignore no oculten robots/llms/sitemap.

FORKS LOCKEADOS:
- F1 (turn 2) meta/head: IN. F2 emisión: @graph centralizado cross-@id. F3 tool: `schema-dts@2.0.0` + render util tipado con escape; sin wrapper; documentar límite @id. F4 sitemap: opinionado; SSR vía endpoint propio; lastmod por serialize; namespaces prune. F5 llms.txt: SHIPPEAR como B2A/agent-readiness vía endpoint prerendered, NO SEO. F6 robots: incluir; tokens verificados; training opt-out documentado/comentado. F7 validación: schema-dts compile-time + manual + script custom exportado a #3 (sin sdtt stale); gate en #3. F8 OG/RSS: out→pointer (OG dinámico satori con constraint Workers; RSS = content concern). F9 i18n/hreflang: solo seam. F10 refs: 5 — `json-ld-graph`, `meta-and-canonical`, `sitemap-config`, `robots-and-llms` (incl. patrón text-endpoint prerender=true compartido), `validation-and-aeo` (validación + framing AEO honesto + tipos rich-result 2026 + deprecación FAQ).

CASO CONTRARIO (doble) — RESUELTO:
- Primario (schema-dts muerto, autorar a mano): DERROTADO — 2.0.0 fresco, ~1.7M dl/sem, zero-runtime. Se mantiene schema-dts. Matiz nuevo: NO valida @id cross-refs → el skill lo dice claro y compensa con validación.
- Secundario (llms.txt cargo-cult SEO): CONFIRMADO como SEO play inútil PERO con uso legítimo B2A/agent-readiness → se shippea con framing honesto, no como palanca SEO.

Seams cross-skill: (1) CSP×JSON-LD (×#2) CERRADO favorable — ld+json exento, #2 no rompe structured data. (2) Validación (→#3): #4 exporta script, #3 lo gatea — seam nuevo limpio. (3) llms.txt×LH13: el audit vive en LH13, aún no en LHCI (#3 corre LH12) → no gateable en #3 hoy, anotado.

Confianza: pins/breaking-changes/tipos schema-dts/comportamiento CSP-data-block/deprecación FAQ+quotes AI-features/postura llms.txt+LH13/límite SSR sitemap/patrón endpoint = VERIFICADO (primaria). INFERIDO/MEDIUM: commit exacto de la fix 3.7.3; path exacto `dist/client` + interacción .assetsignore/_headers; mantenimiento actual de sdtt.

Sandbox suffix activo este chat: `sas_k4m2q9`.

Siguiente: seo-aeo-schema turn 5 (Build) — autoría de SKILL.md + 5 refs (`json-ld-graph`, `meta-and-canonical`, `sitemap-config`, `robots-and-llms`, `validation-and-aeo`); leer md-house-style + skill-author primero; `quick_validate.py` (description ≤1024, sin `<`/`>`, sin `: ` mid-string; frontmatter solo {name, description}); `package_skill.py` → `.skill`; commit atómico del fuente (Git Data API). Re-verificar pins schema-dts 2.0.0 / @astrojs/sitemap 3.7.3 al inicio del build.

### 2026-06-17 — seo-aeo-schema · turn 5 (Build) — HECHO · skill #4 COMPLETA

Autoría + validación + empaquetado + commit atómico del fuente. Bundle = SKILL.md + 5 references. Pins re-verificados contra npm al inicio del build: `schema-dts@2.0.0` (latest, 2026-03-23) y `@astrojs/sitemap@3.7.3` (latest, 2026-05-26). Template de casa leído del repo (`web-security-headers`): SKILL.md frontmatter = name+description; refs = title/summary/last_updated/applies_to.

Archivos (path · bytes · blob sha):
- skills/seo-aeo-schema/SKILL.md · 7178 · 09fd7190c49259c17fc70d4ee69d56f6e7205a5d
- references/json-ld-graph.md · 8642 · 15e3d64116ed480da6cbe35ecd881efaabb4df89
- references/meta-and-canonical.md · 3585 · cfea41391a110104a9f36f18fbf7f3e5b2e1e9a8
- references/robots-and-llms.md · 6299 · 8bbaba8e58074486afcc508350bb58cd2e9f0f61
- references/sitemap-config.md · 4028 · 060ac7de94fb224fa530600cb751e77249a6c8a5
- references/validation-and-aeo.md · 5032 · 04dc549152f3f07da4ed9726db924a5ff0a45117

Validación: `quick_validate` → valid. description 961 chars (≤1024), sin `<`/`>`, sin `: ` mid-string. Frontmatter SKILL.md = {name, description}. House-style: bold=0, HR=0, H4=0 en los 6 archivos; TOC presente en json-ld-graph (149 líneas) y robots-and-llms (143). Las 5 refs parsean YAML OK. Empaquetado: `package_skill` → `.skill` 15750 B con los 6 archivos bajo `seo-aeo-schema/`.

Commit: atómico vía Git Data API (1 commit para todo el bundle). HEAD 33cb7cc → **0b55775** (parent 33cb7cc verificado, autor+committer Carlos). base_tree mergeado → entradas previas del repo preservadas. Verificado por metadata: los 6 archivos existen en skills/seo-aeo-schema/ con los bytes exactos. El `.skill` se entregó a Carlos vía present_files (el repo guarda SOLO las fuentes, no el `.skill`).

Contenido del skill (lo que codifica, resumido): @graph JSON-LD centralizado cross-@id con schema-dts 2.0.0 + render util con escape (`is:inline`, replace `<>&`); doc del límite @id (no se valida → validación runtime obligatoria); detalle CSP-exento (ld+json = data block, sin hash, no tocar scriptDirective.hashes de #2). meta/head typed (canonical derivado de Astro.site+url, OG, Twitter; seam OG dinámico satori/Workers; seam hreflang). @astrojs/sitemap opinionado (solo prerendered; endpoint custom para SSR; lastmod por serialize; bug #16838; changefreq/priority ignorados; dist/client). robots.txt con tabla de 16 tokens verificados + regla acceso≠aparición; llms.txt como B2A/agent-readiness (no SEO) + spec llms.txt vs llms-full.txt; patrón text-endpoint prerender=true compartido. Validación (RRT pierde FAQ jun-2026 vs validator.schema.org) + script CI custom `check-jsonld.mjs` exportado para que el workflow de #3 lo gatee; tabla tipos rich-result 2026 (FAQ deprecado 7-may, HowTo muerto); framing AEO honesto (Google: no special schema; match contenido visible; palancas son content-side).

NOTA DE PROCESO (reusar en builds #5–#7): el commit cross-environment se hizo SIN emitir base64 (regla no-negociable). Método: autorar+validar+empaquetar en code-exec; calcular sha git de blob local de cada archivo; transportar el contenido RAW (no base64) a una celda Composio (backslashes de los snippets de código reemplazados por un centinela `ɵ` y reconstituidos con chr(92) dentro del sandbox para evitar ambigüedad de escape JSON/raw-string); crear blobs con encoding utf-8; GATE: assert sha-de-blob-devuelto == sha-local para los 6 antes de tocar el ref; luego tree(base_tree)+commit+PATCH ref. Los 6 shas matchearon a la primera. Fidelidad probada por metadata, cero base64 en output.

Sandbox suffix activo este chat: `sas_k4m2q9`.

Siguiente: `motion-system` (#5) turn 1 (Scoping). Semilla: 3 motores — GSAP scroll-cinematic / CSS scroll-driven animations (reveals nativos) / Motion (`useAnimate`) para islands React; el fork central es cuándo cada uno y cómo conviven con el presupuesto de JS de #3 y las islands. Pins a re-verificar al lockear: `gsap@3.15.0`, `@gsap/react@2.1.x`, `motion@12.40.0` (antes framer-motion). Cadencia de 5 turnos como siempre; tool-selection no requerido salvo que el scoping lo levante.

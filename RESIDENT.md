---
title: web-stack-skills — RESIDENT (working doc / home base)
updated: 2026-06-18
repo: ccediland/web-stack-skills (público, MIT)
status: cms-self-edit (#8) — COMPLETA. Las 8 skills autoradas; bundle + registro en plugin.json commiteados a `main` (commit `90110a9e`). SIGUIENTE = BUILD FINAL de las 8 skills vía Claude Code (validación + gate de build del marketplace), según `v1-finalization-plan.md`.
---

# web-stack-skills — RESIDENT

Documento vivo y **home base** del proyecto: fuente de verdad única. Reemplaza al plan-de-ejecución original. Contiene qué es, el stack, las 7 skills y sus seeds, la cadencia de autoría, las reglas operativas, las decisiones, los descubrimientos del proceso, el estado y el log de sesiones. Quien lo lea queda al día para continuar.

## 1. Qué es

Un **marketplace de plugins de Claude Code**, público y genérico (MIT), que entrega **8 skills** que codifican un stack web premium de alto rendimiento ("lo mejor de todos los mundos"). Reusable en cualquier proyecto: un proyecto origen fue el primer consumidor, **no** el alcance. Cero contenido específico de proyecto, cero marca, cero secretos. Una 8ª skill queda **diferida** como skeleton.

Meta: parándote en cualquier proyecto nuevo, levantar la misma arquitectura sin volver a descubrir los filos.

## 2. El stack (una herramienta por trabajo)

Astro 6 · Cloudflare Workers Static Assets · Tailwind v4 + Style Dictionary · GSAP + CSS scroll-driven + Motion · OGL · Rive · schema-dts + @astrojs/sitemap + llms.txt · CSP nativo · Lighthouse CI + Biome.

## 3. Las 8 skills + 1 diferida

Orden = fundación → visuales. El seed completo de cada una (veredicto, pins, gotchas, outline) vive en su `SKILL.md`; aquí va el resumen.

| # | Skill | Capa | Veredicto (una línea) |
|---|---|---|---|
| 1 | `astro-css-tokens` | fundación | DTCG `tokens.json` → Style Dictionary **v5** → capa `:root --ds-*` (fuente de verdad) + bridge `@theme inline`; OKLCH sin fallback; en Astro 6 vía `@tailwindcss/postcss` (vite roto, #16542) |
| 2 | `web-security-headers` | fundación | CSP nativo Astro 6 (`security.csp`, hash-based, `<meta>` en estático — CF@13 no soporta `staticHeaders`) + `public/_headers` para HSTS/frame-ancestors/Referrer/Permissions/COOP-CORP (lo que `<meta>` ignora) + middleware solo SSR; SRI manual/astro-shield solo cross-origin; nonce = hash-only |
| 3 | `perf-ci-gates` | fundación | Dos puertas en GitHub Actions — (a) LHCI vía `treosh/lighthouse-ci-action@v12`, `staticDistDir ./dist/client`, preset `no-pwa` + floors (perf 0.9 err / a11y 0.95 err; best-practices+seo warn) + budgets de métrica (LCP/TBT/CLS err) + `budget.json` (KB), INP→TBT proxy, mediana de 5, `temporary-public-storage`+artifacts; (b) Biome `biome ci --reporter=github`, flag `.astro` OFF + `prettier-plugin-astro` (template) + 4 overrides. CWV oficiales SIN cambio (LCP 2.5s/INP 200ms/CLS 0.1) — claim 2.0s FALSO. Gate-only (caching = nota de una pantalla) |
| 4 | `seo-aeo-schema` | fundación | Capa de descubribilidad machine-readable. `schema-dts@2.0.0` (TS-only) → un @graph JSON-LD centralizado cross-@id (Organization↔WebSite[+SearchAction]↔WebPage↔BreadcrumbList↔entidad) emitido inline vía render util con escape; el bloque `application/ld+json` es data block EXENTO del CSP de #2 (no ejecuta → script-src no aplica → no necesita hash). `@astrojs/sitemap@3.7.3` opinionado (solo rutas prerendered; SSR vía endpoint propio; lastmod por serialize). `robots.txt` con tokens AI-crawler verificados (training opt-out = decisión documentada; Googlebot siempre allow; disallow≠fuera-de-respuestas). `llms.txt` como archivo B2A/agent-readiness (Lighthouse 13.3 lo audita) vía endpoint prerendered — NO palanca SEO (Google no lo consume). Componente meta/head typed (canonical/OG/Twitter). Validación = schema-dts compile-time + checks manuales (Rich Results Test + validator.schema.org) + script custom exportado al gate de #3. JSON-LD NO da inclusión en AI Overviews (Google: no special schema) — valor = entity recognition + rich results sobrevivientes. |
| 5 | `motion-system` | visuales | 3 motores por job-fit en escalera native-first — CSS scroll-driven (`animation-timeline: scroll()/view()`, 0 JS / 0 CSP, default de reveals/parallax/progress; `@supports` + estado final accesible; NO Baseline pero production-viable: Safari 26+/Chrome 115+/Firefox-tras-flag, ~83% caniuse) → GSAP+ScrollTrigger (~45 KB gzip; SOLO scrub multi-elemento/pin+scrub/snap/timelines anidadas/motion-path; carga `<script>` bundled o `useGSAP` en isla `client:visible`, lazy off-critical-path; client-only) → Motion solo en islas React existentes (Mini `useAnimate` 2.3 KB imperativo / full `motion/react` 34 KB declarativo). CSP: bundled hasheado por Astro (is:inline bloqueado); `element.style` CSSOM exento de style-src (sin unsafe-inline/hashes; evitar cssText/setAttribute). reduced-motion obligatorio por motor (gate a11y 0.95 de #3). `animation-trigger` discreto = Chrome-only, NO usar |
| 6 | `webgl-atmosfera` | visuales | Atmósfera de hero = un fullscreen-quad fragment shader (fbm domain-warp, ref Alex Harri; highp declarado, loops acotados, probado en móvil). Motor = raw WebGL2 vendorizado (helper 0-dep, 0 KB-lib, WebGL2 universal) default native-first; OGL `1.0.11` Core vendorizado / twgl `7.0.0` alternativas; WebGPU OUT por fitness (1 draw call no usa compute/many-objects; sigue no-Baseline MDN). Carga: canvas NO elegible-LCP (LCP = texto/img) → init requestIdleCallback+IntersectionObserver (NO client:visible above-fold); caja reservada anti-CLS; fade opacity. CSP: módulo bundled hasheado a script-src (paridad #5/#2, experimental.csp astro@6); shaders como template-literals (sin connect-src); contexto canvas no gobernado por CSP. a11y: control pausa/stop visible y operable por teclado OBLIGATORIO (WCAG 2.2.2 nivel A; reduced-motion+aria-hidden NO bastan solos); reduced-motion → fallback estático (cubre 2.3.3 del scroll seam). Fallback: AVIF/WebP estático vía astro:assets (puede ser el img LCP) + gradiente de tokens (#1) secundario; webglcontextlost preventDefault + recrear; sin video. Seam #5: uniform movido por el rAF propio leyendo scroll listener pasivo (0 libs); GSAP solo si la página ya lo carga. Runtime: DPR ≤1.5, ~30fps, pausa IO offscreen + visibilitychange hidden, gate móvil a fallback |
| 7 | `signature-anim` | visuales | Rive (state machine) para UN momento interactivo bespoke; disciplina de integración, no autoría del `.riv`. Renderer `@rive-app/canvas` (Canvas2D, sin límite de contextos WebGL — no compite con #6); `webgl2` solo si Rive Renderer/mesh; nunca `webgl` (legacy). Gate de uso DURO: runtime pesado (~840 KB gzip canvas; lite ~390 KB) — solo si hay lógica de estado ramificada real que CSS/GSAP no codifican y Lottie/dotLottie no igualan; nunca playback/toggle; máx 1 instancia/página. CSP load-bearing: WASM exige `'wasm-unsafe-eval'` en script-src (Astro 6 `security.csp.scriptDirective.resources`, re-listar `'self'`; hashes siguen aditivos) + self-host WASM y `.riv` vía `?url` → `connect-src 'self'` (meta-CSP basta; adapter v13 sin staticHeaders no estorba). Lazy: TS vanilla + IntersectionObserver dynamic-import (no island, no above-fold; canvas no es LCP), reservar caja (CLS), pausar offscreen + visibilitychange, `cleanup()`. Inputs `stateMachineInputs().value`/`.fire()` (NO setNumberState — Android/Unity); Listeners del editor para hover/click; scroll = listener pasivo + rAF a Number input (sin GSAP); `automaticallyHandleEvents:false`. a11y DIY (Rive no trae reduced-motion): WCAG 2.2.2-A exige botón pause si autoplay/loop>5s, 2.3.3-AAA reduced-motion, 2.3.1 flashes. Fallback poster `astro:assets` + try/catch. Evitar `base` no-root (#16276). Bundle 26576412 |

**Diferida — `stack-integration-playbook`** (en `deferred/`, fuera de `skills/`): documenta cómo componen todos los elementos del sitio entre sí y cómo el sitio se conecta al resto del stack más allá de la web (Supabase, otros repos, Cloudflare, GitHub Actions, Infisical, Google Workspace, comms, pagos, catálogos, ads, social, analítica). Se llena "mucho después" con lecciones de campo. Excluida del plugin instalable hasta tener sustancia.

**Skill #8 — `cms-self-edit`** (SELECCIÓN de herramienta HECHA · skill aún sin autorar — su cadencia de 5 turnos arranca en Turn 1 · aún NO en `skills/` ni en `.claude-plugin/`): disciplina para dar a un cliente NO técnico la capacidad de auto-editar el contenido de un sitio Astro 6 + Cloudflare Pages/Workers, sin lock-in tipo Webflow/Framer, como patrón reusable de bajo mantenimiento por sitio. Decisión por MÉRITO puro (sin lente de stack-canon — dropeado por Carlos: no existe / el stack no es fijo). **Veredicto:** ganador = **Sveltia** — CMS git-based client-side puro, montado como `public/admin/index.html`, sin backend; contenido = Markdown/YAML en el repo; OAuth GitHub vía el Worker oficial gratis `sveltia-cms-auth`; media en Cloudflare R2 (SigV4 browser→R2); i18n first-class (DeepL DESHABILITADO en Sveltia — traducción vía Google Cloud Translation / Gemini / Mistral); `/admin` se mantiene como archivo estático en `public/` para esquivar la CSP `<meta>` de #2; publicar vía rama `drafts` + check CI antes de merge (Editorial Workflow aún no es GA). Runner-up = **Pages CMS** — git-based, magic-link por email (el editor NO necesita cuenta GitHub), media R2; pero reintroduce dependencia: app hosteada de terceros con acceso al repo, O self-host con Postgres+BetterAuth. Diseño = **escalera de 3 etapas**: Etapa 1 (default) Sveltia · Etapa 2 (cliente jamás ve GitHub) Pages CMS · Etapa 3 (contenido relacional / multi-canal / publicación instantánea / workflow editorial pesado) Directus (motor por sitio: VPS+Postgres+S3 ~$200/mes; licencia MSCL, OIG libre si <$5M ingresos Y <50 empleados). **Descartados:** Keystatic = roto en Astro 6 (peer-dep `@keystatic/astro@5.0.6` tope astro 2–5, admin truena con error de React hook; issue #1515 ABIERTO, PR #1527 sin merge) + bug OAuth en Cloudflare (#1497 ABIERTO) + sin i18n → review-gate (reconsiderar si #1515 cierra fixed Y #1497 cierra; el cero-i18n sigue siendo tapón MX); EmDash = v0.1.0 dev-preview, contenido en DB/Portable-Text (no archivos), sandbox requiere CF de pago; Tina = pesado (motor+DB o Tina Cloud), Astro experimental, sin i18n nativo; Decap = abandonado (Sveltia es su sucesor); Sanity / headless-DB self-host (Directus/Strapi/Payload/PocketBase) = overkill salvo Etapa 3. Pins a RE-VERIFICAR en build: `@sveltia/cms@0.167.2` (beta, GA mid-2026), `astro@6.x`, `@astrojs/cloudflare@13.x`, Node 22; Worker `sveltia-cms-auth` (gratis) + GitHub OAuth App (`ALLOWED_DOMAINS` = hostname del sitio); bucket R2 + token (Object R/W) + CORS al dominio del CMS + `public_url`. Bundle propuesto: `SKILL.md` (veredicto + escalera + receta Sveltia + gotchas + limits) + references/ {`sveltia-setup`, `cloudflare-auth-worker`, `media-r2`, `escalation-ladder` (Pages CMS + frontera git-vs-DB), `i18n-and-publishing`}. Caveats: Sveltia es beta de un solo mantenedor (riesgo bajo por portabilidad de archivos); la interacción CSP×`/admin` es INFERIDA (verificar en build); Keystatic es time-sensitive (re-chequear). Registro = §11 (selección de herramienta); reporte de research completo = artefacto del chat de decisiones.

### Pins (2026-06-16 — re-verificar en el research de cada skill)

| Skill | Pins |
|---|---|
| astro-css-tokens | `astro@6.4.7`, `tailwindcss@4.3.1`, `@tailwindcss/postcss@4.3.0` (NO `@tailwindcss/vite` — #16542), `style-dictionary@5.4.4` (v5, no v4), Node ≥22.12 — verificar en build |
| web-security-headers | `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`; SRI opc. `@kindspells/astro-shield@1.7.1` (≥1.3.2 por CVE-2024-30250). Review-gate: `staticHeaders` en cada minor de `@astrojs/cloudflare`; CSP no testeable en `dev` (build+preview); watch Astro 7 alpha |
| perf-ci-gates | `@lhci/cli@0.15.1` (LH 12.6.1), `treosh/lighthouse-ci-action@v12`, `@biomejs/biome@2.5.0` (`--save-exact`; conservador `2.4.16`), `prettier-plugin-astro` (solo format de template `.astro`), `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`, **Node 22** (piso Astro 6). Review-gate: thresholds CWV contra web.dev/Search Central (rechazar claim 2.0s); flag `html.experimentalFullSupportEnabled` ON cuando Biome marque HTML estable; LH13 bloqueado de LHCI por Node 22.19+. Verificado 2026-06-17 |
| seo-aeo-schema | `schema-dts@2.0.0` (devDep, TS-only, Schema.org v30; NO valida @id cross-refs — son strings; breaking vs 1.x: Role no-recursivo, Quantity core DataType, dep `schema-dts-lib`), `@astrojs/sitemap@3.7.3` (regresión 3.7.1 #15894 fija; NO ve rutas SSR; bug #16838 lastmod ausente en sitemap-index). Baseline `astro@6.4.7`, `@astrojs/cloudflare@13.7.0` (sin staticHeaders → CSP por `<meta>`; text endpoints requieren `prerender=true`; assets en `dist/client`), Node 22. Review-gates: profundidad de @graph → `…Leaf`/`MergeLeafTypes` (no bajar a 1.1.5); deprecación FAQ (RRT pierde FAQ jun-2026, SC API ago-2026); audit llms.txt de LH13 aún NO en LHCI (#3 corre LH12) |
| motion-system | `gsap@3.15.0` (100% free incl. ScrollTrigger+SplitText, uso comercial; core ~27 KB gzip + ScrollTrigger ~18 KB gzip ≈45 KB), `@gsap/react@2.1.2` (`useGSAP`), `motion@12.40.0` (ex framer-motion, import `motion/react`; Mini `motion/react-mini` useAnimate 2.3 KB). Review-gates: Firefox scroll-driven sin-flag (re-check `layout.css.scroll-driven-animations.enabled`); `animation-trigger` Chrome-only (no usar hasta multi-browser); ScrollTrigger gzip single-source (confirmar en build). Verificado 2026-06-17 |
| webgl-atmosfera | Default = raw WebGL2 vendorizado (helper 0-dep, WebGL2 universal/Baseline). Alternativas: `ogl@1.0.11` (latest verificado; ~1 año sin publish; README sigue "alpha"; 0 deps; Core ~8 KB minzip, subset fullscreen una fracción — tree-shaken sin cifra publicada, medir en build) vendorizando Core subset (Renderer/Triangle/Program/Mesh), sin `^`; `twgl@7.0.0` (mantenido, 0 deps). WebGPU OUT (fitness; MDN no-Baseline may-2026; gpuweb: Firefox Linux/Android pendiente, Safari OS-gated). `astro@6` experimental.csp + scriptDirective.hashes. WCAG 2.2.2 (A) pausa obligatoria + 2.3.3 (AAA) scroll. Review-gates: ogl post-1.0.11 sin "alpha" → reconsiderar dep npm; WebGPU pasa Baseline MDN + se necesita compute → reabrir; set-LCP (canvas excluido) confirmar en build. Verificado 2026-06-18 |
| signature-anim | `@rive-app/canvas@2.38.1` (default) · `@rive-app/canvas-lite@2.37.3` (tren lite va atrás) · `@rive-app/webgl2@2.38.1` · `@rive-app/canvas-single@2.38.0` · `@rive-app/react-canvas@4.28.0` · `@rive-app/webgl` legacy (evitar). Gates: re-pin semanal; medir WASM brotli servido (261 KB@v2.0.0 vs ~640 KB@v2.38.1) y presupuestar contra lo medido; CSP solo build+preview; `?url` bajo Rolldown-Vite; evitar `base` no-root (#16276). Verificado 2026-06-18 |

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
- **El activo durable es el MÉTODO, no las skills** (motor portable: este RESIDENT + cadencia 5-turn + caso contrario + source-priority + log); las skills son perecederas y reconstruibles desde docs. El `stack-integration-playbook` diferido concentra el valor irremplazable (lecciones de composición de campo) — su prioridad de llenado sube en cuanto haya sustancia. (Revisión externa 2026-06-17, adoptado.)
- **Disciplina de perecederos:** cada pin duro y cada workaround por issue-number lleva review-gate; solo + sin auto-update = half-life corto → presupuestar mantenimiento activo, no asumir estabilidad.
- **Capa visual = doble filo** (gana el pitch, arriesga CWV/a11y/lectura B2B): UNA por sitio, cuando el brief la justifique, nunca default.
- **§6 sin cambios** ante la sugerencia externa de smoke-test temprano de install (decisión de Carlos, 2026-06-17: No).
- **Tesis comercial/GTM fuera del repo público** (vive en notas privadas de Carlos); el README puede nombrar el nicho como posicionamiento si Carlos lo decide.
- **Auth de cms-self-edit:** Sveltia default salvo que los clientes rechacen GitHub como norma (fork abierto, llamada de Carlos; sin respuesta, queda Sveltia).

## 8. Descubrimientos del proceso (2026-06-16/17)

- **Layout de plugin** (verificado vs docs oficiales): skills van bajo `skills/<n>/SKILL.md` (no en raíz); hace falta `.claude-plugin/plugin.json`; las skills deben **registrarse** en `plugin.json` vía `"skills":["./skills/..."]` — el auto-discovery solo no basta.
- **Quirk de GitHub** (verificado en ejecución): en repo recién creado **vacío**, la Git Data API (`git/trees`) da 409 "Git Repository is empty" → bootstrapear con un commit inicial (Contents API) y luego reemplazar el árbol completo.
- Poner la 8ª en `deferred/` (fuera de `skills/`) la excluye **estructuralmente** — más limpio que listas de exclusión en el manifiesto.
- **raw.githubusercontent y el render de GitHub sirven copias stale** del repo — leer el estado real vía clon local o Contents API, nunca el raw/render.
- **Commits cross-environment (Git Data API) sin base64:** transferir archivo por archivo (el payload de una sola celda truena ~31 KB), gate sha-git-de-blob == local ANTES de mover el ref, y parentear sobre el HEAD vivo (puede avanzar por commits web entre turnos).

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
- `motion-system` (#5, visuales) turns 1–4 — HECHOS (scoping, pre-research, research verificado fuente primaria, lock). Reframe = árbol de 3 motores native-first (CSS scroll-driven / GSAP scrub / Motion solo-islas). F1–F10 lockeados; veredicto/pins reescritos en §3. Caso contrario doble (GSAP-only ↔ CSS-only, en tensión) derrotado en la frontera de capacidad medida. Seams: CSP×motion favorable (bundled hasheado, is:inline bloqueado, `element.style` CSSOM exento de style-src); peso ~45 KB GSAP vs 2.3 KB Mini × budget #3; reduced-motion por motor × a11y 0.95 #3.
- `motion-system` (#5) turn 5 (Build) — HECHO. Bundle (SKILL.md + 5 refs) autorado, validado (quick_validate OK; description 886 chars; frontmatter solo {name, description}; sin bold/HR/H4; TOC en gsap-astro 112 líneas), empaquetado (.skill 14114 B), commiteado atómico vía Git Data API en `1bd35f1585b8c35bc634b71b3faf4893067c3736` (sha-git de blob == local para los 6 antes de mover el ref). Skill #5 COMPLETA. Detalle en §11.
- `webgl-atmosfera` (#6, visuales) turns 1–4 — HECHOS (scoping/10 forks, pre-research, research verificado fuente primaria, lock). 4 reversals vs leans de turn 1: F1 motor (OGL → raw WebGL2 vendorizado default, native-first), F5 fallback (gradiente CSS → AVIF/WebP estático astro:assets primario), F6 carga (client:visible → init manual rIC+IntersectionObserver; canvas no elegible-LCP), F7 a11y (aria-hidden+reduced-motion → + control pausa OBLIGATORIO SC 2.2.2). CC-A (just-use-CSS) acotado no derrotado; CC-B (OGL estancado) aceptado (movió F1). Seams: CSP×shader paridad #5/#2 (bundled hasheado, template-literals sin connect-src); scroll→uniform por rAF+listener pasivo (0 libs) × #5; reduced-motion × a11y 0.95 #3.
- `webgl-atmosfera` (#6) turn 5 (Build) — HECHO. Bundle (SKILL.md + 5 refs) autorado por md-house-style + skill-author, validado (quick_validate OK; description 951/1024; frontmatter solo {name, description}; sin bold/HR/H4; TOC en los 2 refs >100 líneas), empaquetado (.skill 17707 B, 6 archivos), commiteado atómico vía Git Data API en `22f805c5ca1275f4198ff788e16cd2d7ded74a38` (parent 1881820f; sha-git de blob == local para los 6 antes de mover el ref). Skill #6 COMPLETA. Detalle en §11.
- 6/7 redactadas.
- Siguiente — `cms-self-edit` (#8) turn 1 (Scope) en chat nuevo (cadencia de la skill; selección de herramienta ya hecha); al cierre del turn 5, BUILD FINAL de las 8 skills vía Claude Code (quick_validate + package las 7, probar marketplace add/install/triggering); luego empezar a llenar el deferred `stack-integration-playbook`.

## 10. Roadmap

- **Fase 0** — scaffold del marketplace. Hecha.
- **Skills 1–7** — autoría por la cadencia de 5 turnos, orden fundación → visuales. (7/7 redactadas; #7 `signature-anim` 26576412; #2 `web-security-headers` 4f37a05; #3 `perf-ci-gates` 23d8388f; #4 `seo-aeo-schema` 0b55775; #5 `motion-system` 1bd35f15; #6 `webgl-atmosfera` 22f805c5 — COMPLETA; siguiente = #7 `signature-anim` turn 1)
- **Skill #8 — `cms-self-edit`** — selección de herramienta HECHA (Sveltia default / Pages CMS runner-up / Directus Etapa 3); la cadencia de autoría de la skill (5 turnos) arranca en Turn 1 (Scope); ver §3 y §11. Scaffold al repo se hace en el build (turn 5).
- **Build final (Claude Code)** — `quick_validate` + `package` de las 7, prueba de `marketplace add` / `install` + triggering; después empezar a llenar la skill diferida con lecciones de campo.

## 11. Log de sesiones

Historial compactado en su lugar (2026-08-17, Phase A del sprint v1): cada sesión destilada a qué se hizo, commit y decisión clave. Los verdictos completos viven en §3; decisiones en §7; descubrimientos en §8; el detalle turno-por-turno original queda en la historia de git (pre-compactación).

### 2026-06-16/17 — Fase 0 (scaffold) — hecha
Repo público creado; 13 archivos vía Composio/Git Data API (scaffold `a4bd7f0`, registro de las 7 en `plugin.json` `b24c124`; bootstrap previo por el quirk de repo vacío → §8).

### 2026-06-17 — astro-css-tokens (#1) — COMPLETA · commit `9b182b0`
Decisiones clave: SD v5 + `@tailwindcss/postcss` (NO vite — #16542, review-gate) + bridge `@theme inline` sobre capa `:root --ds-*`; OKLCH sin fallback; `$value` string plano (#1398/#1494); build por prebuild script; dark = costura `[data-theme="dark"]`. Research verificado → §8. Bundle SKILL.md + 5 refs.

### 2026-06-17 — web-security-headers (#2) — COMPLETA · commit `4f37a05`
Pivote verificado en source: el adapter CF 13.x NO soporta `staticHeaders` → CSP estático = `<meta>` hash-based; `frame-ancestors`/report/sandbox → `public/_headers`; middleware solo SSR; hash-only (sin nonce); COEP OFF default (rompe assets cross-origin de #6/#7); SRI = astro-shield ≥1.3.2 solo cross-origin con review-gate. Bundle + 5 refs.

### 2026-06-17 — Revisión externa (dev front-end senior) — registrada (filtrada)
Adoptado → §7 (el método es el activo; el playbook concentra el valor; disciplina de perecederos; visuales doble filo). Smoke-test temprano de install: Carlos decidió No (§6 sin cambios). Tesis comercial/GTM mantenida fuera del repo público.

### 2026-06-17 — perf-ci-gates (#3) — COMPLETA · commit `23d8388f`
Dos puertas: LHCI vía `treosh@v12` (REVERSAL vs autorun crudo — la GitHub App era MÁS integración, no menos), `staticDistDir ./dist/client`, preset no-pwa, floors perf 0.9 / a11y 0.95 error, mediana de 5, TBT proxy de INP ‖ Biome `ci --reporter=github`, flag `.astro` OFF + `prettier-plugin-astro` (REVERSAL — formatter HTML experimental, no matchea Prettier). Claim "CWV LCP 2.0s" confirmado FALSO (2.5/200/0.1 vigentes). Turn 5 recuperado tras corte, sin re-autoría a ciegas.

### 2026-06-17 — seo-aeo-schema (#4) — COMPLETA · commit `0b55775`
Crux desactivado: CSP×JSON-LD = NO-ISSUE (`ld+json` es data block exento de script-src). schema-dts 2.0.0 vivo pero `@id` NO se type-checkea → validación runtime + script exportado al gate de #3. llms.txt = B2A/agent-readiness, NO palanca SEO; robots.txt = surface real de control AI-crawler (16 tokens verificados). Sitemap solo prerendered; SSR vía endpoint propio. Bundle + 5 refs.

### 2026-06-17 — motion-system (#5) — COMPLETA · commit `1bd35f15`
Árbol de 3 motores native-first: CSS scroll-driven default (~83% caniuse, NO Baseline, production-viable) → GSAP+ScrollTrigger solo scrub/pin/snap/timelines anidadas (~45 KB gzip; "7 KB" falso) → Motion solo islas React existentes (Mini 2.3 KB). CSP favorable: bundled hasheado, `element.style` CSSOM exento de style-src. GSAP 100% free verificado (claim "Club" falso). Bundle + 5 refs.

### 2026-06-18 — webgl-atmosfera (#6) — COMPLETA · commit `22f805c5`
4 reversals vs leans de scoping: motor = raw WebGL2 vendorizado (no OGL como dep npm); fallback primario = AVIF/WebP `astro:assets` (no gradiente CSS); carga = init rIC+IntersectionObserver manual (canvas NO elegible-LCP; no `client:visible` above-fold); a11y = control de pausa OBLIGATORIO (WCAG 2.2.2-A). WebGPU OUT por fitness. Runtime: DPR ≤1.5, ~30fps, pausa offscreen. Bundle + 5 refs.

### 2026-06-18 — signature-anim (#7) — COMPLETA · commit `26576412`
Rive Canvas2D para UN momento interactivo bespoke; gate de uso DURO (runtime ~840 KB gzip); CSP load-bearing: `wasm-unsafe-eval` + self-host de WASM y `.riv` → `connect-src 'self'`; lazy vía IntersectionObserver + dynamic-import; a11y DIY (Rive no trae reduced-motion). Proceso: transferencia archivo-por-archivo con gate de sha y parenteo sobre HEAD vivo (→ §8). Bundle + 4 refs.

### 2026-06-18 — cms-self-edit (#8) — selección + skill COMPLETAS · commits `a8c0948` / `90110a9` / `d7da1b2`
Selección por mérito puro (stack-canon dropeado por Carlos como autoridad): Sveltia ganador · Pages CMS runner-up (magic-link) · Directus Etapa 3; Keystatic OUT (#1515/#1497, review-gate), EmDash/Tina/Decap OUT. Build: `/admin` estático en `public/` esquiva la CSP `<meta>`; publicación vía rama `drafts` + CI; correcciones del build promovidas a §3 (DeepL deshabilitado; COOP de `/admin` = `same-origin-allow-popups`; latencia "~2-3 min" sin fuente — medir en sitio real). Con esto 8/8 autoradas.

### 2026-08-17 — Análisis pre-sprint + re-scope del plan v1
Análisis profundo read-only vía Claude Code (11 agentes; 24 claims load-bearing verificados adversarialmente — 23 confirmados, 1 corregido en atribución): estado del repo, re-pin audit completo (→ §3 pins objetivo B0), inventario de `furever-brand`, contrato `brand-system-skills` 0.6.0, auditoría de solape de triggers (3 colisiones ALTAS + cluster medio). Decisiones adjudicadas: migrar a Astro 7 antes de validar (B0); ingestión de marca = skill nueva `brand-canon-ingest` (#9, B1); sitio de referencia = Furever (C); descriptions fix-pass = prerequisito estructural (A). Plan re-scopeado (reemplazo completo de `v1-finalization-plan.md`) commiteado en rama `claude/v1-sprint-rescope`; hallazgos durables → §8.

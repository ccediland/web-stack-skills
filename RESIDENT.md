---
title: web-stack-skills — RESIDENT (working doc / home base)
updated: 2026-08-17
repo: ccediland/web-stack-skills (público, MIT)
status: Roadmap full-catalog en curso (`execution-plan.md`, re-scope 2026-08-18). Phases A/B0/B1 mergeadas (`d1591e0`, `ff6552f`, `cb4f26e`). Phase B2 EJECUTADA — descriptions descongeladas a Astro 7 (7+2 manifiestos), 9/9 validate+package PASS, install real en ambos scopes vía marketplace de path local, triggering 30/30 sin misfires (jueces ciegos), smoke admin Sveltia 0.191.2 PASS, manifiestos en 1.0.0-rc + tag. SIGUIENTE = C: fixture Furever (`furever-web`, deploy temporal Workers, jamás go-live).
---

# web-stack-skills — RESIDENT

Documento vivo y **home base** del proyecto: fuente de verdad única de hechos durables. Contiene qué es, el stack, las skills y sus verdictos, la cadencia de autoría, las reglas operativas, las decisiones, los descubrimientos del proceso, el estado y el log de sesiones. Quien lo lea queda al día para continuar. (El plan mortal del roadmap en curso vive en `execution-plan.md`; cita este doc, nunca lo duplica.)

## 1. Qué es

Un **marketplace de plugins de Claude Code**, público y genérico (MIT), que entrega **9 skills** que codifican un stack web premium de alto rendimiento ("lo mejor de todos los mundos"). Reusable en cualquier proyecto: un proyecto origen fue el primer consumidor, **no** el alcance. Cero contenido específico de proyecto, cero marca, cero secretos. La skill de integración (`stack-integration-playbook`) queda **diferida** como skeleton hasta tener sustancia de campo.

Meta: parándote en cualquier proyecto nuevo, levantar la misma arquitectura sin volver a descubrir los filos.

## 2. El stack (una herramienta por trabajo)

Astro 6 · Cloudflare Workers Static Assets · Tailwind v4 + Style Dictionary · GSAP + CSS scroll-driven + Motion · OGL · Rive · schema-dts + @astrojs/sitemap + llms.txt · CSP nativo · Lighthouse CI + Biome.

## 3. Las 9 skills + 1 diferida

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

**Skill #8 — `cms-self-edit`** (COMPLETA — autorada, validada y registrada en `plugin.json`; commits `a8c0948` selección / `90110a9` bundle): disciplina para dar a un cliente NO técnico la capacidad de auto-editar el contenido de un sitio Astro 6 + Cloudflare Pages/Workers, sin lock-in tipo Webflow/Framer, como patrón reusable de bajo mantenimiento por sitio. Decisión por MÉRITO puro (sin lente de stack-canon — dropeado por Carlos: no existe / el stack no es fijo). **Veredicto:** ganador = **Sveltia** — CMS git-based client-side puro, montado como `public/admin/index.html`, sin backend; contenido = Markdown/YAML en el repo; OAuth GitHub vía el Worker oficial gratis `sveltia-cms-auth`; media en Cloudflare R2 (SigV4 browser→R2); i18n first-class (DeepL DESHABILITADO en Sveltia — traducción vía Google Cloud Translation / Gemini / Mistral); `/admin` se mantiene como archivo estático en `public/` para esquivar la CSP `<meta>` de #2; publicar vía rama `drafts` + check CI antes de merge (Editorial Workflow aún no es GA). Runner-up = **Pages CMS** — git-based, magic-link por email (el editor NO necesita cuenta GitHub), media R2; pero reintroduce dependencia: app hosteada de terceros con acceso al repo, O self-host con Postgres+BetterAuth. Diseño = **escalera de 3 etapas**: Etapa 1 (default) Sveltia · Etapa 2 (cliente jamás ve GitHub) Pages CMS · Etapa 3 (contenido relacional / multi-canal / publicación instantánea / workflow editorial pesado) Directus (motor por sitio: VPS+Postgres+S3 ~$200/mes; licencia MSCL, OIG libre si <$5M ingresos Y <50 empleados). **Descartados:** Keystatic = roto en Astro 6 (peer-dep `@keystatic/astro@5.0.6` tope astro 2–5, admin truena con error de React hook; issue #1515 ABIERTO, PR #1527 sin merge) + bug OAuth en Cloudflare (#1497 ABIERTO) + sin i18n → review-gate (reconsiderar si #1515 cierra fixed Y #1497 cierra; el cero-i18n sigue siendo tapón MX); EmDash = v0.1.0 dev-preview, contenido en DB/Portable-Text (no archivos), sandbox requiere CF de pago; Tina = pesado (motor+DB o Tina Cloud), Astro experimental, sin i18n nativo; Decap = abandonado (Sveltia es su sucesor); Sanity / headless-DB self-host (Directus/Strapi/Payload/PocketBase) = overkill salvo Etapa 3. Pins a RE-VERIFICAR en build: `@sveltia/cms@0.167.2` (beta, GA mid-2026), `astro@6.x`, `@astrojs/cloudflare@13.x`, Node 22; Worker `sveltia-cms-auth` (gratis) + GitHub OAuth App (`ALLOWED_DOMAINS` = hostname del sitio); bucket R2 + token (Object R/W) + CORS al dominio del CMS + `public_url`. Bundle propuesto: `SKILL.md` (veredicto + escalera + receta Sveltia + gotchas + limits) + references/ {`sveltia-setup`, `cloudflare-auth-worker`, `media-r2`, `escalation-ladder` (Pages CMS + frontera git-vs-DB), `i18n-and-publishing`}. Caveats: Sveltia es beta de un solo mantenedor (riesgo bajo por portabilidad de archivos); la interacción CSP×`/admin` es INFERIDA (verificar en build); Keystatic es time-sensitive (re-chequear). Registro = §11 (selección de herramienta); reporte de research completo = artefacto del chat de decisiones.

**Skill #9 — `brand-canon-ingest`** (COMPLETA — autorada, validada, empaquetada y registrada en `plugin.json`; commit `4e76123`): disciplina de CONSUMO de un repo de marca emitido por brand-system-skills (contrato `0.6.0` @ tool-repo `abcc31f`, pin con review-gate sobre las 4 superficies consumidas; los repos emitidos no llevan stamp de versión — el contrato lo auto-imponen sus gates copiados) hacia un sitio del stack. **Veredicto:** el repo de marca es la única fuente de verdad y el sitio una PROYECCIÓN registrada; la skill es la disciplina del cruce. Precondición paso 0 no negociable: `node tools/run-gates.mjs` del repo de marca en verde (NOT-RUN honesto ≠ FAIL). Tokens: `tokens/web/{base,semantic,component}.json` copiados VERBATIM como source de SD — pipeline delegado a #1, jamás apuntar SD al spine estructurado (`*.tokens.json`); +1 transform sancionado `value/cubic-bezier-css` (cubicBezier llega como array DTCG — crudo es CSS inválido; verificado contra SD 5.5.1 real). Schemes: serializador C-1 vendorizado (~50 líneas zero-dep, byte-parity con `tokens-project.mjs` del repo de marca) emite `schemes.css` con vars a nivel rol — `:root`=default light PRIMERO (empate de especificidad: el orden de emisión decide la cascada), variantes opt-in por clase, `[data-theme="dark"]` como seam del stack + bridge semántico para que las utilities Tailwind reaccionen al scheme; default LIGHT sin auto-switch (G-UX-02) — no-flash SIN `prefers-color-scheme`, divergencia deliberada del script de #1. Assets: logos exact-file (G-LOGO-01; `-fc` OFF-SYSTEM por G-LOGO-02) · iconos currentColor (re-tintan con el scheme gratis) · `@font-face` SOLO woff2 shippeados + `font-synthesis-weight: none` + family name byte-igual al stack del token · favicon del iso-mono · OG raster (G-IMG-03) vía receta playwright 1200×630 on-token. Voz: resident set (canon.json + AMBOS keystones juntos + asset-index como mapa); truth-gating de claims a fuente; citar reglas por ID `G-*`/`ALGO-*`, jamás restatearlas (fork de verdad). Gobernanza: registrar la proyección en `satellites/projections.md` (fila machine-checked R6a — pin de valor stale REVIENTA el board de la marca); data pointers (volátiles jamás congelados; catalog→DB mismo contrato); protocolo root→gates→re-ingest (jamás parchar site-side); promotion path para decisiones nacidas en el sitio (owner ratifica, el sitio no se auto-promueve). Gotchas extra: group-level `$type` es DTCG válido (no "arreglar" los archivos copiados); tokens JS-vocabulary (press-scale, thresholds, `px/s`) por import JSON, no CSS vars; compat aliases legacy solo para consumers viejos. Evidencia dura: serializador corrido contra `furever-brand` real — 212 vars (53×4 schemes), orden correcto, C-1 normalizado sin hex. Bundle: SKILL.md + 5 refs (scheme-serializer, brand-repo-map, assets-fonts-og, voice-and-copy, governance-projections). Description 975 chars con frontera bilateral (builder/scoper = construir ≠ consumir; #1 = pipeline delegado, "tokens.json plano sin repo de marca" es #1); contraparte en la description de #1 (commit `b19b693`).

### Pins por skill (histórico 2026-06-16 — donde choque, manda la tabla vigente de abajo; el detalle vivo post-B0 está en cada SKILL.md)

| Skill | Pins |
|---|---|
| astro-css-tokens | `astro@6.4.7`, `tailwindcss@4.3.1`, `@tailwindcss/postcss@4.3.0` (NO `@tailwindcss/vite` — #16542), `style-dictionary@5.4.4` (v5, no v4), Node ≥22.12 — verificar en build |
| web-security-headers | `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`; SRI opc. `@kindspells/astro-shield@1.7.1` (≥1.3.2 por CVE-2024-30250). Review-gate: `staticHeaders` en cada minor de `@astrojs/cloudflare`; CSP no testeable en `dev` (build+preview); watch Astro 7 alpha |
| perf-ci-gates | `@lhci/cli@0.15.1` (LH 12.6.1), `treosh/lighthouse-ci-action@v12`, `@biomejs/biome@2.5.0` (`--save-exact`; conservador `2.4.16`), `prettier-plugin-astro` (solo format de template `.astro`), `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`, **Node 22** (piso Astro 6). Review-gate: thresholds CWV contra web.dev/Search Central (rechazar claim 2.0s); flag `html.experimentalFullSupportEnabled` ON cuando Biome marque HTML estable; LH13 bloqueado de LHCI por Node 22.19+. Verificado 2026-06-17 |
| seo-aeo-schema | `schema-dts@2.0.0` (devDep, TS-only, Schema.org v30; NO valida @id cross-refs — son strings; breaking vs 1.x: Role no-recursivo, Quantity core DataType, dep `schema-dts-lib`), `@astrojs/sitemap@3.7.3` (regresión 3.7.1 #15894 fija; NO ve rutas SSR; bug #16838 lastmod ausente en sitemap-index). Baseline `astro@6.4.7`, `@astrojs/cloudflare@13.7.0` (sin staticHeaders → CSP por `<meta>`; text endpoints requieren `prerender=true`; assets en `dist/client`), Node 22. Review-gates: profundidad de @graph → `…Leaf`/`MergeLeafTypes` (no bajar a 1.1.5); deprecación FAQ (RRT pierde FAQ jun-2026, SC API ago-2026); audit llms.txt de LH13 aún NO en LHCI (#3 corre LH12) |
| motion-system | `gsap@3.15.0` (100% free incl. ScrollTrigger+SplitText, uso comercial; core ~27 KB gzip + ScrollTrigger ~18 KB gzip ≈45 KB), `@gsap/react@2.1.2` (`useGSAP`), `motion@12.40.0` (ex framer-motion, import `motion/react`; Mini `motion/react-mini` useAnimate 2.3 KB). Review-gates: Firefox scroll-driven sin-flag (re-check `layout.css.scroll-driven-animations.enabled`); `animation-trigger` Chrome-only (no usar hasta multi-browser); ScrollTrigger gzip single-source (confirmar en build). Verificado 2026-06-17 |
| webgl-atmosfera | Default = raw WebGL2 vendorizado (helper 0-dep, WebGL2 universal/Baseline). Alternativas: `ogl@1.0.11` (latest verificado; ~1 año sin publish; README sigue "alpha"; 0 deps; Core ~8 KB minzip, subset fullscreen una fracción — tree-shaken sin cifra publicada, medir en build) vendorizando Core subset (Renderer/Triangle/Program/Mesh), sin `^`; `twgl@7.0.0` (mantenido, 0 deps). WebGPU OUT (fitness; MDN no-Baseline may-2026; gpuweb: Firefox Linux/Android pendiente, Safari OS-gated). `astro@6` experimental.csp + scriptDirective.hashes. WCAG 2.2.2 (A) pausa obligatoria + 2.3.3 (AAA) scroll. Review-gates: ogl post-1.0.11 sin "alpha" → reconsiderar dep npm; WebGPU pasa Baseline MDN + se necesita compute → reabrir; set-LCP (canvas excluido) confirmar en build. Verificado 2026-06-18 |
| signature-anim | `@rive-app/canvas@2.38.1` (default) · `@rive-app/canvas-lite@2.37.3` (tren lite va atrás) · `@rive-app/webgl2@2.38.1` · `@rive-app/canvas-single@2.38.0` · `@rive-app/react-canvas@4.28.0` · `@rive-app/webgl` legacy (evitar). Gates: re-pin semanal; medir WASM brotli servido (261 KB@v2.0.0 vs ~640 KB@v2.38.1) y presupuestar contra lo medido; CSP solo build+preview; `?url` bajo Rolldown-Vite; evitar `base` no-root (#16276). Verificado 2026-06-18 |

### Pins vigentes (2026-08-17 — re-pin audit verificado adversarialmente; APLICADOS en B0)

| Paquete | Pin previo | Vigente | Nota |
|---|---|---|---|
| `astro` | 6.4.7 | **7.2.2** | 7.0.0 stable desde 2026-06-22; `security.csp` sobrevive el major (solo aditivo — opción `kind` en 7.1.0) |
| `@astrojs/cloudflare` | 13.7.0 | **14.2.1** | peer `astro ^7.2.0` — migración acoplada; `staticHeaders` sigue SIN aterrizar en el adapter (review-gate de #2 NO dispara); watch #16692 — hashed assets sin Cache-Control immutable en algunas configs (reportado en 13.5.0), re-verificar bajo 14.2.1 en B0/C |
| `tailwindcss` | 4.3.1 | **4.3.3** | + **switch-back a `@tailwindcss/vite`** — review-gate #16542 dispara con Astro 7 (verificado empíricamente); la ruta `@tailwindcss/postcss` pasa a nota histórica |
| `style-dictionary` | 5.4.4 | **5.5.1 — JAMÁS 5.5.0** | 5.5.0 = GHSA-xmr7-549p-98w3 (prototype pollution HIGH; npm audit puede no verla). #1398 cerrado desde 5.4.0; #1494 abierto → workaround string solo para typography |
| `@biomejs/biome` | 2.5.0 | **2.5.8** | patch-only; flag `.astro` sigue experimental — se mantiene "OFF + prettier-plugin-astro" |
| `@rive-app/canvas` / `@rive-app/webgl2` | 2.38.1 | **2.40.0** | webgl2 −11.5% de tamaño (Emscripten 4.0.23 + -Os) |
| `@sveltia/cms` | 0.167.2 | **0.191.2** | 24 minors adelante; pre-GA (milestone 1.0 RC cerca) → smoke-test del admin PENDIENTE en B2 |
| `motion` | 12.40.0 | **13.1.0** | único breaking = @emotion/is-prop-valid (no nos toca); trae `animateView` (View Transitions) |
| `@kindspells/astro-shield` | 1.7.1 | **RETIRADA** | abandonada (último publish nov-2024, peer astro ^4) → reemplazada por la receta SRI sha384 custom en #2 (B0-2) |
| gsap · @gsap/react · schema-dts · @astrojs/sitemap · @lhci/cli · treosh@v12 | — | **sin cambio** | todos siguen latest; #16838 (lastmod sitemap-index) ya está fijo en el pin 3.7.3 |

Gotchas y outline detallados — en cada `skills/<nombre>/SKILL.md`.

## 4. Estructura del repo

    .claude-plugin/
      marketplace.json     -> 1 plugin: web-stack (source ".")
      plugin.json          -> manifiesto + "skills":[ las 9, rutas relativas ]
    skills/
      astro-css-tokens/SKILL.md
      web-security-headers/SKILL.md
      perf-ci-gates/SKILL.md
      seo-aeo-schema/SKILL.md
      motion-system/SKILL.md
      webgl-atmosfera/SKILL.md
      signature-anim/SKILL.md
      cms-self-edit/SKILL.md
    deferred/
      stack-integration-playbook/SKILL.md   (excluido del plugin)
    README.md  LICENSE  RESIDENT.md  CLAUDE.md  execution-plan.md

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

- **Superficie por época** — en la fase de autoría (2026-06) GitHub fue TODO por Composio (Git Data API) y Claude Code quedó reservado al build final. Desde el sprint v1 (2026-08-17), Claude Code ES la superficie de ejecución (ver Surfaces del plan); Composio queda como vía alternativa para commits desde chat.
- **Un plugin** (`web-stack`) agrupa las 9. Instalación: `/plugin marketplace add ccediland/web-stack-skills` luego `/plugin install web-stack@web-stack-skills`.
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

### 2026-08-17 — Hallazgos pre-sprint (verificados contra fuentes primarias; re-checados adversarialmente)

- **Astro 7.2.2 stable** (7.0.0 el 2026-06-22); `@astrojs/cloudflare@14.2.1` exige `astro ^7.2.0` — migración acoplada. `security.csp` sobrevive el major (cambios aditivos). Mina activa en Astro 6: npm puede hoistear Vite 8 junto a Astro 6 y romper el build hoy (mitigación temporal: overrides `{"vite":"^7"}`). Node 20 se dropeó desde Astro **6.1.0**, no en 7.
- **2 citas erróneas en `astro-css-tokens`** (corregir en B0): Astro 6 nunca usó rolldown-vite — el bug real era npm hoisteando Vite 8 (pnpm/yarn no afectados); el "#19802" citado es de tailwindlabs/tailwindcss, no de rolldown-vite. Firefox scroll-driven: sigue tras flag en stable (Nightly 156 default ON; soporte global ~85.4%; watch ~oct-2026).
- **furever-brand ya habla el contrato:** `tokens/web/{base,semantic,component}.json` es proyección string pre-emitida para SD v5 — se consume tal cual. Delta real: los 4 schemes (dark + estados, 53 roles c/u) son objetos OKLCH compuestos SIN proyección string → serializar con el patrón C-1 de `tools/tokens-project.mjs` (~50 líneas zero-dep) a bloques CSS de override a nivel rol; default light sin auto-switch (G-UX-02); overrides al tier semántico, no al base. Menores: cubicBezier arrays → envolver en `cubic-bezier()`; tokens JS-vocabulary (px/s, rootMargin) por import JSON; favicon derivable de `furever-iso-mono-*.svg`; OG raster obligatorio (G-IMG-03).
- **Contrato brand-system-skills 0.6.0:** spine de máquina fijo + gates auto-enforced (suite copiada a cada repo emitido); web-stack-skills es el consumidor flagship (e2e verificado 2026-07-16; proyección `furever-web` ya registrada en `satellites/projections.md`). NO garantizado: schemes, component tier, fuentes bundleadas, ni stamp de versión de contrato dentro del repo emitido — el hop C-1→web ya se rompió ~6 semanas una vez → la skill #9 pinea el contrato (0.6.0 + commit del tool-repo) con review-gate y corre `run-gates.mjs` del repo de marca como precondición.
- **Superficie de triggers de las 8:** 3 colisiones ALTAS (CSP motion↔security; inversión léxica "hash-based CSP" — 4 hermanas la nombran, la dueña no; "animated gradient background" sin hook léxico en motion-system) + cluster medio (reduced-motion, pause, lazy-load, scroll-driven, perf-budget); 4 descriptions sin frontera; cero redirects entre hermanas. Fix-pass aplicada en Phase A; el eje "design tokens"/"brand" de la #9 quedó RESUELTO en el triggering de B2 (abajo). Probes de B1 y su desenlace: (a) "set up design tokens for our new brand" — el juez ciego de B2 ruteó CORRECTO a `brand-canon-builder` (margen narrow; el ancla fue "for our new brand" + el disclaimer de palette de #1); sin fix local posible sin inflar — registrada como UPSTREAM SUGGESTION a brand-system-skills (§9); (b) "dark mode" — hook léxico añadido a #1 en B2 ("add dark mode through token overrides", 750 chars).

### 2026-08-17 — Lecciones de B2 (validación mecánica)
- **Triggering con jueces ciegos escala y es reproducible:** 30 prompts × 10 agentes independientes que solo ven las 11 descriptions (9 nuestras + builder/scoper) → 30/30 sin misfires; margins narrow señalan fronteras reales aunque ruteen bien ("hero background" webgl-vs-motion por diseño; "make my site feel premium" → NONE, que es lo sano — ningún catálogo debe capturar un prompt puramente estético). Batería y metodología reusables para cada ola (W1–W4).
- **Install de prueba sin push:** `claude plugin marketplace add <path-local> --scope user|project` registra el checkout como marketplace `source: directory` y LEE EN VIVO — la vía canónica para validar el estado de una rama antes de merge. `--scope project` escribe `.claude/settings.json` en el cwd (por eso el `.gitignore` nuevo cubre `.claude/`). Verificación: `claude plugin details web-stack@web-stack-skills` lista las 9 skills.
- **Sveltia 0.167→0.191 (24 minors): receta intacta** — bundle npm montado en `public/admin/` bootea, parsea `config.yml` (schema actual sin breaking) y muestra los 3 flujos de auth. El riesgo pre-GA sigue siendo cadencia, no breakage.
- Ecosistema de pins (detalle en §3 pins objetivo B0): Sveltia 0.191.2 pre-GA con cadencia casi diaria (single-maintainer sano; sveltia-cms-auth activo); Keystatic review-gate NO cumplido (#1515 cerró solo peer-dep vía PR #1534; #1497 sigue abierto); GSAP/`@gsap/react` sin release nuevo y 100% free confirmado; LH13 NO ha aterrizado en LHCI (issue #1136 sin respuesta); DTCG 2025.10 sigue siendo el único stable.

### 2026-08-17 — Lecciones del smoke scaffold B0 (build real Astro 7)

- **`@tailwind base` es no-op SILENCIOSO en Tailwind v4:** el build PASA pero Tailwind nunca procesa el CSS — cero utilities generadas y la línea literal llega al browser. El entry stylesheet debe abrir con `@import 'tailwindcss'`; los bloques `@theme` se recolectan de todo el import graph. (Cazado por el smoke; corregido en astro-css-tokens.)
- Confirmado en build real 7.2.2 + adapter 14.2.1 + `@tailwindcss/vite` 4.3.3 + SD 5.5.1: CSP meta hash-based en página estática ✓ · utilities compilan a cadenas `var(--ds-*)` ✓ · OKLCH preservado (lightningcss solo reformatea L a porcentaje) ✓ · dark override ✓ · el adapter SÍ inyectó `Cache-Control` immutable para `/_astro/*` en la config default (watch #16692: OK aquí; re-verificar en el deploy real de C).
- Astro 7 ahora avisa a nivel config que Shiki × CSP chocan (consistente con la guía Prism de #2).

## 9. Estado

- Skills #1–#9: autoradas y commiteadas (commit por skill en §11). 9/9 registradas en `plugin.json`.
- Phase A (cleanup/docs): MERGEADA a main (PR #1, merge `d1591e0`).
- Phase B0 (migración Astro 7): MERGEADA a main (`ff6552f`) — pins vigentes en §3 (astro 7.2.2, adapter 14.2.1, `@tailwindcss/vite` switch-back, SD 5.5.1 jamás 5.5.0, biome 2.5.8, motion 13.1.0, rive 2.40.0, sveltia 0.191.2, astro-shield RETIRADA), smoke scaffold Astro 7 verde con 1 bug de receta cazado y corregido (Tailwind entry → §8).
- Phase B1 (`brand-canon-ingest` #9): MERGEADA (`cb4f26e`) — bundle + registro + frontera bilateral en #1; serializador y transform verificados contra `furever-brand` real.
- Phase B2 (validación mecánica): EJECUTADA en rama `claude/b2-mechanical-validation` — plan reemplazado por `execution-plan.md` (re-scope full-catalog del owner); descriptions descongeladas a Astro 7 (7 skills — el censo real, no 6 — + ambos manifiestos); 9/9 `quick_validate` + `package_skill` PASS (scripts canónicos de `anthropics/skills@main`, `skills/skill-creator/scripts/`); install real en scopes user y project vía marketplace de path local (lee el checkout en vivo — §8); triggering 30/30 con jueces ciegos; smoke Sveltia 0.191.2 PASS; manifiestos a `1.0.0-rc`.
- UPSTREAM SUGGESTION registrada (a formalizar en W5, repo `brand-system-skills`): la description de `brand-canon-builder` ganaría un ancla explícita tipo "set up design tokens for a NEW brand (no existing token source)" — hoy el prompt rutea bien pero con margen narrow vs `astro-css-tokens`; el fix vive de su lado, no del nuestro. (Se suma a las dos ya anotadas en el plan W5: stamp de contrato en repos emitidos; proyección de schemes emitida por el builder.)
- Gobernanza post-sprint del owner (fuera del plan, por directiva 2026-08-18): ratificación Stage-10 del canon v2 · GAP-006 banco de imagen · GAP-014 aviso de privacidad · doc licencia BTN. El fixture usa placeholders.
- Siguiente: C — fixture Furever en `furever-web`: componer el subconjunto, decidir acceso al repo de marca (fork abierto), CI gates, deploy TEMPORAL a subdominio Workers.

## 10. Roadmap

### Roadmap en curso (full catalog — `execution-plan.md`)
Fase 0 + skills #1–#9: HECHAS. El roadmap corre por `execution-plan.md` (doc mortal, re-scope 2026-08-18 a catálogo completo): B2 validación mecánica → C fixture Furever → D playbook (ship 1.0.0) → olas W1–W4 (backlog completo, minor por ola) → W5 capa cliente (v2.0.0). Decisiones adjudicadas 2026-08-17/18: migrar antes de validar; ingestión de marca = skill nueva; Furever = fixture permanente (deploys temporales, jamás go-live); pull-by-project retirado (desviación deliberada del owner — Lego sobrevive en composición).

### Principio Lego (regla permanente del catálogo)
El bundle es un catálogo, no un sistema fijo. Cada sitio compone solo el SUBCONJUNTO que su brief necesita; ningún sitio lleva todas las skills; la capa visual entra UNA vez, cuando el brief la justifica, nunca por default. Una skill nueva amplía el alcance del catálogo, no el payload de cada sitio.

### Backlog (asignado a olas W1–W5 en `execution-plan.md`; pull-by-project RETIRADO por el owner 2026-08-18 — la tabla conserva los build-triggers como contexto histórico)
Prioridad comprometida: **`client-discovery`** (intake/descubrimiento del cliente) — único ítem v2 especificado. Job: convertir lo que el cliente dé, en el formato que sea (escrito / dibujo / export design-tool), en un brief estructurado y validado contra el stack, + registro de lo diferido a Carlos. 4 fases: intake (banco de preguntas/plantillas) → captura por formato (escrito→requisitos; dibujo→requisitos+handoff visual; export→pipeline de tokens) → factibilidad (corre contra la matriz del playbook; veredicto por ítem, límites, delta brief↔deploy) → deferral (separa estético vs funcional/UX; registra como decisión de Carlos con contexto). Fronteras: captura intención, NO renderiza ni diseña; la factibilidad vive en el playbook (referencia, no duplica); diferidos se registran. Depende de Phase D (playbook); gate de arranque: v1 shipped. Nombre final al autorarla.

| Skill candidata | Tier | Build-trigger |
|---|---|---|
| `data-layer` (datos externos, filtros/búsqueda, live data, lógica de negocio) | 1 | un proyecto pide menú/catálogo/datos dinámicos — sesgar a build-time (candidato natural: el sitio Furever en Phase C) |
| `forms-lead-system` (o pick de herramienta + receta en playbook) | 1 | un proyecto necesita captura/leads/contacto (candidato: Furever en C) |
| a11y profundo (más allá del piso 0.95 de perf-ci-gates) | 2 | un cliente exige WCAG AA formal |
| `i18n-system` | 2 | un proyecto multi-idioma real |
| `media-optimization` | 2 | un sitio image/video-heavy lo exige |
| `edge-logic` (Workers: A/B, redirects, geo, feature flags) | 2 | un proyecto necesita lógica en el edge |
| `analytics-measurement` (privacy-friendly + eventos) | 2 | un cliente pide medición real |
| view-transitions / conversion-patterns / content-modeling / speculation-rules / auth-simple / legal-compliance / component-scaffolding / visual-regression-ci | 3 | especulativas — solo si un proyecto las jala |
| `stack-governance` | ✗ | NO es skill — la cadencia ya hace review-gates; vive como disciplina de este RESIDENT (§7) |

Invariantes de toda skill v2: native-first (declarar qué opción nativa se descartó y por qué) · una herramienta por trabajo, sin solape · review-gate en cada pin duro y cada workaround por issue-number · description filosa y sin solape (la superficie de discoverability) · buy/configure sobre custom-build (custom = excepción documentada) · regla Lego.

Drift honesto (postura registrada): el setup-and-forget puro NO aplica a los sitios construidos sobre esto — esperar ciclos de toque de ~12–18 meses (majors de Astro, cambios del adapter). Mitigaciones: sitios tan estáticos como el brief permita; `data-layer` sesgada a build-time; review-gates en cada pin; el playbook documenta los seams frágiles. B0 (Astro 7) es el primer ciclo real — presupuestar los siguientes.

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

### 2026-08-17 — Phase B2 (validación mecánica) — EJECUTADA · rama `claude/b2-mechanical-validation`
Bajo el plan nuevo (`execution-plan.md`, re-scope full-catalog 2026-08-18 del owner; reemplaza y elimina `v1-finalization-plan.md`). B2-1 descongelamiento: 7 descriptions "Astro 6"→"Astro 7" (censo real: 7, no 6 — webgl seguía en 6 por el revert deliberado de B0) + recorte seo-aeo-schema 1023→1015 + fix "v13" stale en #2; después del triggering, hook "dark mode" en #1 (750). B2-2: 9/9 validate+package (scripts canónicos descargados de `anthropics/skills@main`). B2-3 install: marketplace de path local, scopes user y project, 9 skills descubiertas (`claude plugin details`). B2-4 triggering: 30 prompts, 10 jueces ciegos, 30/30 — todos los obligados OK; probe cross-plugin ruteó correcto (narrow) → upstream suggestion (§9). B2-5 smoke Sveltia 0.191.2: PASS (screenshot verificado; banner de incidente GitHub real confirmó red viva). B2-6: manifiestos 1.0.0-rc + descriptions de manifiesto a Astro 7. Lecciones durables → §8.

### 2026-08-17 — Phase B1 (brand-canon-ingest #9) — EJECUTADA · rama `claude/b1-brand-canon-ingest`
Scope lockeado en chat; verificación de terreno contra `furever-brand` real ANTES de autorar: `run-gates.mjs` ALL-GREEN (19 PASS + 1 NOT-RUN honesto — red-team live diferido por diseño), 4 schemes × 53 roles, contrato 0.6.0 @ `abcc31f`. Cero reversiones de arquitectura; 3 ajustes menores documentados (orden de emisión del serializador — empate de especificidad, `:root` primero o dark gana en light; fila de projections es machine-checked R6a; cita exacta del veto `-fc` = G-LOGO-02). Bundle #9 autorado bajo skill-author + md-house-style, validado (`quick_validate` PASS) y empaquetado (`.skill` NO commiteado); evidencia dura de los 2 artefactos de código (serializador 212 vars contra repo real; transform cubicBezier compilado en SD 5.5.1). Frontera brand añadida a la description de #1 (711 chars, diferido de A3 saldado). Misroute check contra 8 hermanas + 2 plugins brand-system: sin canibalización bidireccional. Ratificación Stage-10 NO registrada (el prompt no abrió con la frase-mecanismo) — queda con Carlos. Commits: `4e76123` (bundle+registro) · `b19b693` (frontera #1) · docs.

### 2026-08-17 — Phase B0 (migración Astro 7) — EJECUTADA · rama `claude/b0-astro7-wave`
Phase A mergeada (PR #1, `d1591e0`). Re-pin lockstep aplicado skill por skill, un commit c/u: #1 astro 7.2.2 + tailwind 4.3.3 + switch-back a `@tailwindcss/vite` (#16542 cerrado; 2 citas corregidas) + SD 5.5.1 (jamás 5.5.0) · #2 adapter 14.2.1 (gate staticHeaders NO disparó) + astro-shield RETIRADA → receta SRI sha384 custom + watch #16692 · #3 biome 2.5.8 (flag `.astro` sigue OFF) · #5 motion 13.1.0 + Firefox ~85.4% (watch Fx156 ~oct-2026) · #7 rive 2.40.0 · #8 sveltia 0.191.2 (gate Keystatic NO cumplido) · baseline 7.2.2/14.2.1 en #4/#6. Smoke scaffold Astro 7 completo compiló y verificó CSP/tokens/utilities/dark/Cache-Control; 1 bug de receta cazado y corregido (Tailwind v4 entry — no-op silencioso, → §8).

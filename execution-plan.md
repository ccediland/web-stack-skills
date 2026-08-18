---
name: web-stack-full-catalog
description: "Execution plan Claude works from to take ccediland/web-stack-skills from 9-skills-authored to the complete catalog shipped — mechanical validation, the Furever integration fixture, the integration playbook, five capability waves covering the entire backlog (data, forms, i18n, media, a11y, edge, analytics, full tier 3), and the client-facing layer culminating in client-discovery and v2.0.0. Use when resuming this work in a fresh chat."
title: "web-stack-skills — Full Catalog Roadmap (v1.0.0 → v2.0.0)"
summary: "Mortal execution plan, re-scoped 2026-08-18 by owner directive: the full backlog is BUILT, not declared — pull-by-project retired as a deliberate deviation; Lego survives at composition time. Furever is a permanent integration fixture (temp Workers deploys at gates, never a final build). Sequence: B2 validation → C fixture → D playbook (ship 1.0.0) → W1–W4 capability waves (1.1–1.4) → W5 client layer (2.0.0). Points at RESIDENT.md for standing facts; archived at v2 ship."
last_updated: 2026-08-18
applies_to: "ccediland/web-stack-skills · post-B1 (9 skills authored, Astro 7 baseline) · consumes ccediland/furever-brand as test fixture (contract 0.6.0)"
status: "IN PROGRESS — W3 Run B EXECUTED and v1.3.0 SHIPPED: edge-logic as thin skill #15 (locks applied verbatim, zero reversals) + analytics recipe in the playbook (canon corrected: Umami = events default, CF WA = ambient RUM), fixture extended LIVE (client-side A/B verified E2E in a real browser; analytics wiring shipped as an HONEST STUB, env-gated, delivery not claimed), triggering 27/28 first-pass + 1 real lexical gap fixed and re-tested over a 17-description surface. W4 Run A EXECUTED: full tier-3 triage (10 candidates, every one with a home verdict) with adversarial verification of 14 load-bearing claims."
phase: "W4 (tier 3) — Run A triage delivered, AWAITING CHAT LOCK of home verdicts → then Run B (build the wave → v1.4.0)"
home_base: "chat (claude.ai) for adjudication only; all execution and analysis in Claude Code (Fable 5)"
next_action: "Chat: lock the W4 home verdicts from the Run A triage table (3 thin skills: auth-simple · visual-regression-ci · conversion-patterns / 4 absorptions / 2 playbook recipes / 1 updated boundary). Then Code: W4 Run B — build per verdicts, extend the fixture where it applies, ship v1.4.0"
resident: "./RESIDENT.md — canonical for all standing facts (skill verdicts, pins, decisions, state). Point, never duplicate."
---

# web-stack-skills — Full Catalog Roadmap

> Plan de ejecución re-scopeado (2026-08-18) por directiva del dueño: construir e integrar TODO el catálogo — core, backlog completo y capa cliente — hasta v2.0.0. Doc mortal; hechos durables en `RESIDENT.md`. Reemplaza a `v1-finalization-plan.md` (renombrado: su nombre ya mentía).

## TL;DR
- Meta final: el marketplace completo — 9 skills actuales + playbook + 7 skills de olas W1–W3 + tier 3 completo con hogar (W4) + `client-discovery` y recipes (W5) — validado, instalable, con el fixture Furever como banco de pruebas permanente. v1.0.0 al cerrar D; minor por ola; v2.0.0 al cerrar W5.
- Desviación deliberada registrada (owner, 2026-08-18): la regla "skill jalada por proyecto real" se retira — el catálogo se pre-construye completo. El principio Lego sobrevive donde importa: EN COMPOSICIÓN — cada sitio sigue componiendo un subconjunto; ninguna skill entra a un sitio por default.
- Furever = FIXTURE: repo de marca real usado como insumo de prueba. Builds de integración y deploys TEMPORALES a subdominio de Workers en los gates; jamás build final, jamás go-live. GAP-006/GAP-014/licencias/ratificación Stage-10 = gobernanza post-sprint del dueño, fuera de este plan.
- Cadencia por ola (contexto de chat al mínimo): Run A en Code (scope+research+decision sheet de la ola completa) → chat lockea forks en UN turno → Run B en Code (build+validate+triggering+fixture+tag+merge). Playbook se expande al cierre de cada ola.
- Régimen operativo vigente: Code auto-mergea tras verificación adversarial pre-merge; pausa solo ante irreversibles (borrados, force-push, releases públicos mayores). Un commit por unidad; ramas `claude/<fase>`; docs (RESIDENT + este plan) al cierre de cada run.

## When to use this document
En un chat fresco durante el trabajo: lee el state block + front matter y §3/§8/§9/§10 del RESIDENT, confirma fase y next action con Carlos, ejecuta, loguea incrementalmente, reescribe estado + session log al cierre. No reiniciar ni re-derivar lo resuelto.

## Objective — three horizons
- **H1 — v1.0.0 (core probado):** las 9 skills validadas e instalables (B2) + fixture Furever ejercitando los seams reales (C) + `stack-integration-playbook` v1 con sustancia de campo (D). Done-criteria en Phase D.
- **H2 — v1.x (catálogo completo):** olas W1–W4 — todo el backlog construido, cada skill validada contra el fixture, un minor release por ola. Done-criteria por ola.
- **H3 — v2.0.0 (capa cliente):** `client-discovery` + composition recipes por arquetipo + madurez de marketplace (versionado, calendario de drift, stamp de contrato upstream). Cierra el roadmap; este plan se archiva.

## This document's lifespan
Execution plan (species 1, ai-roadmap): estado mutable de sesión. RESIDENT = fuente de verdad durable; este plan CITA, no copia. Cada ola migra sus verdictos/pins/decisiones a RESIDENT §3/§7/§8 en su Run B. Al shippear v2.0.0: archivar.

## The Lego principle — revised standing
El bundle es un catálogo completo por decisión; la composición sigue siendo por subconjunto. Invariantes que TODA skill nueva mantiene: native-first (declarar qué opción nativa se descartó) · una herramienta por trabajo, sin solape · review-gate en cada pin duro y workaround · description filosa con fronteras y redirects a hermanas · buy/configure sobre custom (custom = excepción documentada) · la skill amplía el catálogo, no el payload de cada sitio.

## Full catalog map (target end-state)
| Capa | Skills |
|---|---|
| Fundación (hechas) | astro-css-tokens · web-security-headers · perf-ci-gates · seo-aeo-schema |
| Visuales (hechas) | motion-system · webgl-atmosfera · signature-anim |
| Contenido & marca (hechas) | cms-self-edit · brand-canon-ingest |
| Composición | stack-integration-playbook (D; vive y crece por ola) |
| W1 Data & captura | data-layer · forms-lead-system |
| W2 Alcance & pulido | i18n-system · media-optimization · a11y-deep |
| W3 Edge & medición | edge-logic · analytics-measurement |
| W4 Tier 3 (triage: skill / absorción / recipe) | view-transitions · speculation-rules · conversion-patterns · content-modeling · auth-simple · legal-compliance · component-scaffolding · visual-regression-ci · (+ huérfanas: hover micro-interactions, Lottie-boundary) |
| W5 Cliente | client-discovery · composition recipes · marketplace maturity |

## Phase B2 — Mechanical validation (9 skills) → v1.0.0-rc
Goal: probar que el bundle instala y dispara.
- Descongelar las 6 descriptions "Astro 6" → baseline Astro 7 (cambio mínimo; seo-aeo-schema a 1023/1024: recortar antes de tocar). Tabla antes/después.
- `quick_validate.py` + `package_skill.py` canónicos sobre las 9; arreglar fallos.
- Install real: `/plugin marketplace add` + `/plugin install web-stack@web-stack-skills`, ambos scopes; documentar la vía (estado mergeado vs checkout local).
- Triggering test ≥25 prompts: ejes parchados en A3 (CSP, backgrounds), eje de #9 (3 obligados + probe cross-plugin "set up design tokens for our new brand" — si el fix exige tocar brand-system-skills, registrar como upstream suggestion, no tocar), prompts sin tecnología ("hero background", "make my site feel premium"). Misfire → fix → re-test.
- Smoke del admin Sveltia 0.191.2 (salto 24 minors); arreglar receta si rompió.
- Tag v1.0.0-rc (anotado); plugin.json → 1.0.0-rc.
Gate B2→C: 9/9 validate PASS · install limpio ambos scopes · triggering sin canibalización · smoke Sveltia OK · rc taggeado.

## Phase C — Furever integration fixture
Goal: ejercitar los seams DE VERDAD con la marca real como insumo de prueba. Code autónomo (no es fase de aprendizaje de Carlos — eso vuelve cuando haya sitio real de cliente).
- Repo del sitio: `furever-web` (proyección ya registrada). Componer el subconjunto que el brief-fixture pida: brand-canon-ingest + tokens + CSP + perf-gates + schema; motion; UN visual; cms-self-edit.
- Decidir y documentar el acceso al repo de marca (clone hermano vs submodule vs checkout CI) — primera decisión real de composición, va al playbook.
- Seams a ejercitar: ingest→tokens load order · schemes/dark · hidratación · CSP × JSON-LD × GSAP bundled · budget × motion · reduced-motion × a11y 0.95 · CMS · deploy WSA.
- CI gates (LHCI + Biome) contra el build real. Deploy TEMPORAL a subdominio Workers (workers.dev o subdominio de prueba del dueño) para verificación viva en el gate; puede quedarse como staging del fixture o derribarse — nunca es go-live.
- Capturar cada gotcha cross-cutting al momento (por referencia en session log; detalle al playbook en D). Documentar el subconjunto compuesto (primera composition recipe).
- Contenido faltante (imágenes, aviso, licencias): PLACEHOLDERS marcados — es fixture.
Gate C→D: fixture compila · pasa ambas puertas CI · preview desplegado y verificado · lecciones capturadas.

## Phase D — stack-integration-playbook v1 → SHIP v1.0.0
Goal: promover la diferida a skill real desde las lecciones de C; shippear v1.
- Cadencia comprimida (research = lecciones de C): scope/decisiones → build. Contenido: cómo componen las skills (load order, hidratación, seams ejercidos, cómo compone brand-canon-ingest), gotchas cross-cutting, integración web ↔ resto del stack, primera recipe (Furever-fixture).
- Mover de `deferred/` a `skills/`; registrar en plugin.json; validar; triggering vs hermanas.
- Ship v1.0.0: bump, tag release, README con el fixture como ejemplo + install verificado, RESIDENT al día.
Done-criteria v1.0.0: release taggeado · install + triggering limpios de las 10 piezas · fixture pasando ambas puertas CI con preview verificado · playbook en `skills/` con sustancia real.

## Waves W1–W4 — capability build-out (v1.1.0 → v1.4.0)
Cadencia por ola: **Run A (Code)** = scoping + pre-research + research verificado fuente primaria de TODAS las skills de la ola + decision sheet destilada (forks con leans + evidencia) → **chat lockea en un turno** → **Run B (Code)** = build de los bundles + validate/package + registro + triggering (batería que incluye TODAS las hermanas previas) + extensión del fixture Furever para ejercitar la ola + expansión del playbook + docs + tag minor + merge. Toda skill cumple los invariantes Lego (arriba). Deploy temporal del fixture cuando el gate de la ola lo pida.

**W1 — Data & captura → v1.1.0**
- `data-layer`: datos externos, filtros/búsqueda/interactividad, lógica de negocio; sesgo BUILD-TIME (native-first), live solo justificado; seam natural con el canon (data-map → catalog.json → Supabase — el fixture Furever ya lo pide por diseño).
- `forms-lead-system`: captura/leads/contacto; decidir en Run A si es skill propia o pick stack-canon + recipe (lean del plan viejo: resolver con evidencia).
- Fixture: extender con catálogo sintético + form de contacto de prueba.
Gate: 2 skills (o 1+recipe) validadas · fixture ejercitándolas · triggering limpio contra 10 previas · v1.1.0.

**W2 — Alcance & pulido → v1.2.0**
- `i18n-system`: multi-idioma (routing, tokens de contenido, hreflang × seo-aeo-schema).
- `media-optimization`: imagen/video pesados (astro:assets a fondo, formatos, LCP × perf-ci-gates).
- `a11y-deep`: WCAG AA formal más allá del piso 0.95 (seams con motion/reduced-motion y visuales).
- Fixture: segunda locale sintética + set de media de prueba.
Gate: 3 skills validadas · fixture ejercitándolas · triggering limpio · v1.2.0.

**W3 — Edge & medición → v1.3.0**
- `edge-logic`: Workers (A/B, redirects, geo, feature flags) — frontera dura con web-security-headers (middleware) y perf.
- `analytics-measurement`: privacy-friendly + eventos (Cloudflare Web Analytics / PostHog per stack-canon; seam CSP connect-src).
- Fixture: un experimento A/B sintético + eventos de prueba.
Gate: 2 skills validadas · fixture · triggering · v1.3.0.

**W4 — Tier 3 completo, con triage → v1.4.0**
Cada candidata recibe en Run A un veredicto de HOGAR (nada queda al aire): (a) skill propia, (b) absorción en hermana existente (extensión versionada de esa skill), o (c) recipe del playbook. Leans iniciales — Run A los confirma o revierte con evidencia:
- `view-transitions` → lean absorción en motion-system (motion 13 trae animateView) o skill propia si el alcance lo rebasa.
- `speculation-rules` → lean recipe de perf (playbook o perf-ci-gates).
- `conversion-patterns` → lean skill propia (CRO estructural, localizable MX).
- `content-modeling` → lean absorción en cms-self-edit o recipe.
- `auth-simple` → lean skill propia delgada (Cloudflare Access / Workers, native-first).
- `legal-compliance` → lean recipe del playbook (aviso de privacidad MX, cookies, LFPDPPP) — contenido legal se marca como plantilla, no asesoría.
- `component-scaffolding` → lean absorción en astro-css-tokens/playbook.
- `visual-regression-ci` → lean skill propia (seam directo con perf-ci-gates; /shot ya existe en el tooling).
- Huérfanas: hover micro-interactions → motion-system (extensión); Lottie → frontera documentada (Rive es el pick; una herramienta por trabajo) sin skill.
Gate: TODAS las candidatas con hogar ejecutado (skill construida / extensión mergeada / recipe escrita) · triggering total del catálogo · fixture donde aplique · v1.4.0.

## W5 — Client layer & marketplace maturity → v2.0.0
- `client-discovery`: 4 fases (intake → captura por formato → factibilidad contra la matriz del playbook → deferral registrado). Fronteras: captura intención, no renderiza (→ Claude Design); factibilidad VIVE en el playbook (referencia). Nombre final se decide en su Run A.
- Composition recipes por arquetipo (corporativo-imagen, landing-premium, restaurante+catálogo, storytelling, consultoría-AI) — al playbook.
- Marketplace maturity: política de versionado escrita (semver del plugin, lockstep skills↔manifest) · calendario de drift (watches vivos: Firefox 156 ~oct-2026, Sveltia GA, LHCI×LH13, majors Astro ~12–18m) como sección del RESIDENT · upstream suggestions formalizadas a brand-system-skills (stamp de versión de contrato en repos emitidos; proyección de schemes emitida por el builder).
- Ship v2.0.0: bump, tag, README final, RESIDENT (status: catálogo completo), archivar este plan.
Done-criteria v2.0.0: catálogo completo instalable · triggering total limpio · playbook con recipes · client-discovery operativa · política de mantenimiento escrita.

## Maintenance & drift (standing, post-v2 home = RESIDENT)
Ciclos de toque ~12–18 meses (majors Astro, adapter). Cada release de ola re-verifica pins de las skills que toca; los watches viven en RESIDENT §3. B0 fue el primer ciclo real — presupuestar los siguientes, no asumir estabilidad.

## Open questions / forks
- Acceso del sitio al repo de marca — RESUELTO en C: vendorización en ingest-time (lock de chat, sostuvo contra la evidencia; CI standalone).
- W1 — RESUELTO Y LOCKEADO (chat, 2026-08-17; ejecutado en Run B tal cual): data-layer = loader build-time default / file() tier cero / client-side escalación / live fuera; rebuild = webhook→Deploy Hook directo (repository_dispatch variante documentada); typing = zod único vía astro/zod; llaves nuevas + tombstone @astrojs/db. forms = RECIPE del playbook (flip #3 resuelta por el check: Email Service ganó el slot; #1/#2/#4 siguen vigentes como condiciones de promoción a skill).
- W2 — RESUELTO Y LOCKEADO (chat, 2026-08-17; ejecutado en Run B tal cual, cero reversiones): i18n = skill propia (prefixDefaultLocale false · diccionario zero-dep con Sveltia single_file como perilla · multiple_folders + translationKey/canonical_slug · SIN detección (flip "main ya cargado" documentado, NO ejercido en fixture) · hreflang = mapa de i18n-system EMITIDO por la superficie head de seo-aeo-schema, frontera bilateral en ambas); media = skill delgada (compile explícito · constrained+responsiveStyles · priority solo LCP · video en tiers · CF Images escape hatch); a11y = skill propia (@axe-core/playwright 4.13.0 · todas las built + muestra WCAG-EM manual · zero-violations con target-size habilitado · checklist dos tiers · seam contract en perf-ci-gates). Todo ejercitado en vivo por el fixture (tercera puerta axe incluida).
- W3 — RESUELTO Y LOCKEADO (chat, 2026-08-17; ejecutado en Run B tal cual, cero reversiones): edge-logic = skill DELGADA (A/B client-side · zone rules → custom entry, middleware NON-OPTION en prerendered · geo Intl/endpoint request.cf · flags build-time + Flagship solo-Worker, KV nunca · _redirects default + escalera); analytics = RECIPE del playbook (Umami default de eventos · CF WA = RUM ambiental solo zona proxied · lead_submit server-side · directo sobre proxy · sin banner + analytics en el aviso · GA4 tombstone con excepción Ads).
- Veredictos de hogar de W4 — Run A ENTREGADA (2026-08-17), PENDIENTE de lock: 3 skills propias delgadas (auth-simple — ancla Worker-level Access 2026-08-14 · visual-regression-ci — Playwright snapshots native-first, Lost Pixel muerto, BackstopJS estancado · conversion-patterns — estructural es-MX con frontera "sin datos no hay CRO"), 4 absorciones (view-transitions→motion-system · content-modeling→cms-self-edit · component-scaffolding→astro-css-tokens · hover→motion-system), 2 recipes del playbook (speculation-rules · legal-compliance), 1 frontera actualizada (Lottie/Rive: exports Rive a Cadet $9, dotLottie v2 state machines sin paridad). Decision sheets en el reporte al chat.
- Nombre final de client-discovery — Run A de W5.
- Subdominio de prueba — RESUELTO en C: preview nativo de rama en workers.dev (alias estable por rama, sin auth local).
- Resueltos (2026-08-18): catálogo completo se construye (owner) · Furever = fixture permanente, deploys temporales, sin go-live · ratificación Stage-10 = gobernanza post-sprint, fuera del plan.

## Next actions
1. Chat: lockear los veredictos de hogar de W4 desde la tabla de triage del Run A (3 skills con sus decision sheets · 4 absorciones · 2 recipes · 1 frontera).
2. Code (W4 Run B): construir la ola completa según veredictos lockeados (skills + extensiones versionadas + recipes + frontera); triggering total del catálogo; fixture donde aplique; tag v1.4.0. Probablemente + W5 Run A (client-discovery scoping).

## Resume
Standing instruction: en chat fresco — leer state block + RESIDENT (front matter, §3, §8, §9, §10), confirmar fase y next action con Carlos, ejecutar, loguear incrementalmente, cerrar con estado + session log + hand-off. No reiniciar el plan.

## Session log
### 2026-08-17 — W3 Run B ejecutada (SHIP v1.3.0) + W4 Run A (triage tier 3) — Claude Code
Locks de W3 aplicados tal cual, cero reversiones. `edge-logic` #15 delgada (SKILL.md único: modelo routing/billing, 4 trampas, forks e1–e5, 3 snippets, pins de plataforma; description 1007) + `analytics-recipe.md` en el playbook (Umami + eventos + lead_submit server-side + CSP + env-gate + aviso; pesos medidos; tombstone GA4; corrección del canon; triggers de promoción verbatim); playbook description con eje analytics (1014). Fixture: A/B client-side verificado E2E en browser contra el deploy (0 errores de consola) + analytics stub HONESTO (env-gated, ausencia asertada por el smoke); 3 puertas verdes con anti-vacuo. Triggering 27/28 primera pasada (superficie 17, 7 jueces) + gap léxico RUM cazado → fix → re-test 3/3. Install 15 skills OK. Ship v1.3.0. W4 Run A: 3 agentes de research + verificador adversarial (14 claims); triage completo con hogar para las 10 candidatas; hallazgos: Worker-level Access (2026-08-14), Lost Pixel muerto, Rive exports de pago, reglamento LFPDPPP overdue.
Siguiente: chat lockea veredictos de W4 → Run B.

### 2026-08-17 — W2 Run B ejecutada (SHIP v1.2.0) + W3 Run A — Claude Code
Locks de W2 aplicados tal cual, cero reversiones. Las 3 skills (#12–#14) autoradas desde los briefs del Run A (recuperados verbatim del transcript — cero re-derivación); descriptions 1001/1011/1017; fronteras cruzadas en seo-aeo-schema (1012) y perf-ci-gates (1019) re-juecedas. Fixture: /en/ sintética con hreflang/x-default verificados EN EL DEPLOY (curl + browser E2E 0 errores de consola), blog multiple_folders+translationKey con Sveltia i18n declarado, media set bajo compile explícito (20 derivados webp/avif), TERCERA puerta axe ("scanned 8 pages, 0 violations" en el log) y LHCI a 6 URLs ("Checking assertions against 6 URL(s), 30 total run(s)"). Gotcha de campo nuevo → skill: AxeBuilder exige browser.newContext(). Triggering 28/28 (7 jueces ciegos, superficie 16; fronteras nuevas rutean en ambas direcciones; "add hreflang tags" → seo-aeo-schema NARROW = la bilateral). W3 Run A: 2 briefs con fuentes primarias + verificación adversarial; decision sheet al chat. Ship v1.2.0.
Siguiente: chat lockea forks de W3 → Run B.

### 2026-08-17 — W1 Run B ejecutada (SHIP v1.1.0) + W2 Run A — Claude Code
Locks aplicados tal cual, cero reversiones. Check Email Service: GANA (public beta, verified-destination gratis todos los planes) → slot de la recipe; Resend = flip. `data-layer` 11ª (validate 11/11); forms recipe propia en el playbook con flip #3 resuelta; description del playbook re-autorada (996). Fixture: graduación seed→Supabase probada por contenido (acentos), /contacto vivo con Action+Turnstile (POST 200 en deploy; browser E2E cero errores CSP), ciclo webhook→Deploy Hook end-to-end verificado (~2.5 min), gates verdes con 4 URLs asertadas. Gotchas de campo nuevos → recipe: split-config wrangler/Vite (configPath) y `cloudflare:workers` env (locals.runtime.env removido). Triggering 24/24 (el ambiguo de D ahora es CLEAR → data-layer). W2 Run A: 3 briefs verificados adversarialmente (corrección: imageService default cambió en adapter v13, no v14); check local: contrato 0.6.0 SIN multi-locale (voz = regla es-MX). Ship v1.1.0.
Siguiente: chat lockea forks de W2 → Run B.

### 2026-08-17 — Phase D ejecutada (playbook → SHIP v1.0.0) + W1 Run A — Claude Code
D en cadencia comprimida (research = lecciones de C, cero re-derivación): playbook promovido vía `git mv` con historia, autorado (SKILL.md orden canónico como cadena de dependencias; refs seams/mapa/recipes), registrado 10º, validate 10/10, triggering 16/16 con 4 jueces ciegos sobre 12 descriptions (4 composición → playbook CLEAR; ambiguo → NONE sano), manifiestos 1.0.0, README v1 (fix: brand-canon-ingest faltaba en la tabla), merge + tag `v1.0.0` + GitHub Release. W1 Run A (solo análisis): 2 agentes de research con fuentes primarias + verificador adversarial sobre claims load-bearing; decision sheet destilada al chat. Hallazgos que cambian el terreno: Workers Builds YA tiene Deploy Hooks (2026-04) — el camino nativo de rebuild es webhook de Supabase → hook directo; live collections estables pero fuerzan on-demand (frontera dura del deploy shape); Astro Actions estables con RPC desde páginas prerendered (solo el endpoint es server) — el "sitio full-estático + un endpoint de forms" es first-class; `@astrojs/db` muerta (removida en Astro 7); Supabase migró a `sb_publishable_`/`sb_secret_`.
Siguiente: chat lockea forks de W1 → Run B.

### 2026-08-17 — Phase C ejecutada (fixture Furever) — Claude Code
Deviations del lock con evidencia: repo YA existía (producción viva en main) → fixture en rama `claude/c-fixture`, NUNCA merge; deploy temporal = preview nativo de rama de Workers Builds (workers.dev, sin wrangler auth local). Compuesto: 8 skills (webgl por evidencia — cero `.riv` + ALGO-ATMOSPHERE-COMPOSE). Seams ejercitados de verdad: 84 violaciones CSP por style-attrs → clases → 0; COOP duplicado en `/admin/` cazado EN VIVO → detach `!`; hash manual single-sourced del no-flash; dist/server vacío → assets-only; #16692 no dispara (verificado vivo); GSAP pin + Sveltia + dark switch + data-pointer OK. Gates CI verdes; projections R6a PASS (`furever-brand@5d99526`); preview `claude-c-fixture-furever-web.carlos-872.workers.dev`. 20 lecciones (RESIDENT §8) + 3 fixes de recetas (78b92e5). Resueltos ambos forks abiertos: acceso = vendorización ingest-time (lock sostuvo); subdominio = workers.dev nativo.

### 2026-08-17 — Phase B2 ejecutada (validación mecánica) — Claude Code
Plan reemplazado en-repo (viejo eliminado, referencias actualizadas). B2-1: 7 descriptions descongeladas a Astro 7 (censo real 7, no 6) + recorte seo (1015) + "v13" stale + hook dark-mode post-triggering en #1. B2-2: 9/9 validate+package (scripts canónicos anthropics/skills@main). B2-3: install user+project vía `claude plugin marketplace add <path-local>` (source directory, lee el checkout en vivo — sin push); 9 skills descubiertas. B2-4: 30 prompts, 10 jueces ciegos → 30/30, cero misfires; obligados todos OK; probe cross-plugin correcto (narrow) → upstream suggestion registrada (RESIDENT §9), brand-system-skills NO tocado. B2-5: smoke Sveltia 0.191.2 PASS (receta intacta). B2-6: manifiestos 1.0.0-rc + tag anotado. Detalle durable en RESIDENT §8/§9/§11.
Siguiente: C (fixture Furever).

### 2026-08-18 — Re-scope a catálogo completo (owner directive)
Hecho: adjudicación en chat del rumbo total. Decidido (owner, tras una ronda de pushback registrada): (1) TODO el backlog se construye — pull-by-project retirado como desviación deliberada; Lego sobrevive en composición; (2) Furever = fixture permanente — builds de integración + deploys temporales a subdominio Workers en gates, jamás final/go-live; gates de contenido y ratificación → gobernanza post-sprint; (3) roadmap a 3 horizontes: v1.0.0 (B2→C→D), v1.1–v1.4 (olas W1–W4, minor por ola), v2.0.0 (W5); (4) cadencia por ola Run A/lock/Run B — 2 turnos de chat por ola; (5) plan renombrado a execution-plan.md; (6) W4 con triage de hogar: skill/absorción/recipe — nada queda sin hogar.
Dead-ends: n/a este turno.
Siguiente: B2 (Code) bajo el plan nuevo.

### 2026-08-17/18 — Fases A, B0, B1 (compactado)
A (PR #1, merge d1591e0): plan re-scopeado v1, RESIDENT compactado 656→218, durables migrados, descriptions fix-pass (C1–C12), CLAUDE.md creado, manifiestos+README. B0 (merge ff6552f): ola Astro 7 completa — astro 7.2.2, adapter 14.2.1, @tailwindcss/vite 4.3.3 (gate #16542 cerrado), SD 5.5.1 (jamás 5.5.0), biome 2.5.8, motion 13.1.0, rive 2.40.0, sveltia 0.191.2; astro-shield retirada (SRI custom); smoke scaffold compiló; gotcha @import 'tailwindcss'. B1 (merge cb4f26e): brand-canon-ingest (#9) autorada — run-gates ALL-GREEN sobre furever-brand, serializador C-1 verificado 0-diffs/212 valores, description 975 con fronteras bilaterales, frontera "brand" agregada a astro-css-tokens, 16/17 misroutes OK (probe cross-plugin diferido a B2). Ratificación Stage-10 NO registrada (mecanismo no disparó) → gobernanza post-sprint.

## Limitations & Out-of-Scope
- El fixture Furever NUNCA llega a build final ni go-live; contenido faltante = placeholders.
- Gobernanza de contenido del dueño (ratificación, imágenes, aviso, licencias) fuera del plan.
- Tesis comercial / GTM en notas privadas del dueño.
- Cambios a brand-system-skills solo como upstream suggestions registradas — otro repo, otro plan.

## References
- [RESIDENT.md](./RESIDENT.md): fuente de verdad durable. Apuntar, no duplicar.
- [plugin.json](./.claude-plugin/plugin.json) / [marketplace.json](./.claude-plugin/marketplace.json): manifiestos.
- `skills/<nombre>/SKILL.md`: verdicto, pins, gotchas por skill.
- `~/proyectos/furever-brand` (fixture, contrato 0.6.0) · `~/proyectos/brand-system-skills` (tool-repo del contrato) · `furever-web` (repo del sitio-fixture, Phase C).

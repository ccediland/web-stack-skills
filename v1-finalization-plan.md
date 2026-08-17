---
name: web-stack-v1-finalization
description: "Execution plan Claude works from to take ccediland/web-stack-skills from 8-skills-authored to v1 shipped — cleanup, Astro 7 migration wave, brand-canon-ingest (#9), mechanical validation, the Furever reference site, the integration playbook, and release — plus the v2 Lego roadmap. Use when resuming this sprint in a fresh chat."
title: "web-stack-skills — v1 Sprint (Finalization + Brand Ingestion) & v2 Roadmap"
summary: "Mortal execution plan, re-scoped 2026-08-17 from the pre-sprint deep analysis (11 agents, adversarially verified). Seven gated phases: A cleanup/docs, B0 Astro 7 migration wave, B1 author brand-canon-ingest, B2 mechanical validation of 9 skills, C Furever reference site, D integration playbook, E ship v1. Points at RESIDENT.md for standing facts; archived at v1 ship."
last_updated: 2026-08-17
applies_to: "ccediland/web-stack-skills · post-8-skills · pre-v1-ship · consumes ccediland/furever-brand via brand-system-skills contract 0.6.0"
status: "IN PROGRESS — re-scope adjudicado en chat 2026-08-17; pendiente commit de este plan (rama claude/) y arranque de Phase A."
phase: "A (Cleanup & docs) — lista para arrancar tras el commit de este plan"
home_base: "chat (claude.ai); excursions to Claude Code for A (execution), B0, B1-build, B2, C"
next_action: "Commit de este plan re-scopeado en rama claude/v1-sprint-rescope + Phase A completa (ver Next actions)"
resident: "./RESIDENT.md — canonical for all standing facts (skill verdicts, pins, decisions, state). Point, never duplicate."
---

# web-stack-skills — v1 Sprint & v2 Roadmap

> Plan de ejecución re-scopeado (2026-08-17) para el sprint que lleva el repo de "8 skills redactadas, cero sitios" a "v1 shippeada con capacidad de ingestión de marca y el sitio Furever vivo como prueba". Doc mortal (estado de sesión); los hechos durables viven en `RESIDENT.md`. Reemplaza el scope del 2026-06-18 — el análisis pre-sprint (reporte 2026-08-17, verificado adversarialmente) reordenó las fases.

## TL;DR
- Objetivo H1 (este sprint): v1 = 8 skills existentes + `brand-canon-ingest` (#9, nueva) + `stack-integration-playbook`, todas sobre **Astro 7**, instalables y validadas, con el **sitio Furever** vivo en Cloudflare pasando ambas puertas de CI, construido consumiendo `ccediland/furever-brand` directo.
- Secuencia con compuertas: A cleanup/docs → B0 migración Astro 7 → B1 autoría de #9 → B2 validación mecánica de las 9 → C sitio Furever → D playbook → E ship.
- Tres decisiones que fija este re-scope (adjudicadas 2026-08-17): (1) **migrar a Astro 7 antes de validar** — no shippear v1 un major atrás; (2) la ingestión de marca es **skill nueva** `brand-canon-ingest`, no extensión de `astro-css-tokens` ni recipe del playbook; (3) el sitio de referencia **es el sitio Furever** — integration test + primera corrida real de #9 + activo comercial, tres pájaros.
- Caveat (se repite a propósito): plan MORTAL. Pins, verdictos y decisiones durables viven en `RESIDENT.md`. Phase A migra ahí lo durable de este plan. Al shippear v1, este plan se archiva.
- Principio Lego intacto: cada sitio compone un SUBCONJUNTO. El sitio Furever compone lo que su brief pida — la capa visual entra una vez, si el brief la justifica.
- v2 sigue jalado por proyecto real. Si el sitio Furever jala `forms-lead-system` o `data-layer` (probable: datos volátiles → Supabase por diseño del canon), esa skill se promueve entonces, bajo su cadencia — no se pre-construye.

## When to use this document
Úsalo en un chat fresco durante el sprint. Al abrirlo: lee el state block del front matter + front matter y §3/§9/§10 del RESIDENT, confirma la fase y la next action con Carlos, ejecuta, y al cerrar reescribe estado + añade al session log. No reiniciar el plan ni re-derivar lo que el log ya resolvió.

## Objective
**H1 — v1 shipped:** las 9 skills (8 + `brand-canon-ingest`) + `stack-integration-playbook`, migradas a Astro 7, instalables, con install y triggering validados en Claude Code, y el sitio Furever vivo en Cloudflare pasando LHCI + Biome, construido desde `furever-brand`. Done-criteria en Phase E.

**H2 — v2 (post-sprint):** capa cliente-facing culminando en `client-discovery` (depende del playbook de Phase D). Backlog Lego jalado por proyectos reales.

## This document's lifespan
Execution plan (species 1 de ai-roadmap): estado mutable de sesión. `RESIDENT.md` es la fuente de verdad de hechos durables — este plan los CITA, nunca los copia. Phase A termina la migración de piezas durables al RESIDENT (roadmap v2, principio Lego, "stack-governance no es skill", + los hallazgos durables del reporte pre-sprint).

## The Lego principle
El bundle es un catálogo, no un sistema fijo. Cada sitio compone solo las skills que su brief necesita; una skill nueva amplía el alcance del catálogo, no el payload de cada sitio. `brand-canon-ingest` es exactamente eso: amplía el catálogo con "consumir un repo de marca del builder", y solo entra en sitios que tengan canon de marca.

## v1 coverage of the non-negotiables
| Innegociable | Skill de v1 que lo cubre |
|---|---|
| Performance | `perf-ci-gates` (+ disciplina de budget de `motion-system`) |
| SEO / AEO | `seo-aeo-schema` |
| Visuales | `motion-system` + `webgl-atmosfera` + `signature-anim` (uno, cuando el brief lo pida) |
| Ciberseguridad | `web-security-headers` |
| Alineación de marca | `brand-canon-ingest` (#9 — este sprint) |

## Key inputs (pre-sprint report, 2026-08-17 — durable copies land in RESIDENT at Phase A)
Hechos verificados que gobiernan el sprint. Detalle completo: reporte pre-sprint en el chat home-base del 2026-08-17; lo durable se migra a RESIDENT §8 en Phase A.
- **Astro 7.2.2 stable** (7.0.0 el 2026-06-22); adapter `@astrojs/cloudflare@14.2.1` exige `astro ^7.2.0` — migración acoplada. `security.csp` sobrevive el major (cambios aditivos). Review-gate #16542: con Astro 7, **volver a `@tailwindcss/vite`** (verificado empíricamente). Mina activa en Astro 6: npm puede hoistear Vite 8 y romper el build hoy (mitigación temporal: overrides `{"vite":"^7"}`).
- **`style-dictionary`**: bump a 5.5.1, JAMÁS 5.5.0 (GHSA-xmr7-549p-98w3, prototype pollution HIGH). #1398 cerrado (dimension objects OK); #1494 (typography) abierto — workaround string solo para typography.
- **`@kindspells/astro-shield` efectivamente abandonado** (peer astro ^4) → reemplazar SRI (pieza custom pequeña) DENTRO de la migración B0, no después.
- **Sveltia 0.191.2** (24 minors adelante del pin), pre-GA (RC cerca); bump + smoke-test. Keystatic: gate NO cumplido, seguir Sveltia. Rive 2.40.0 (re-pin, −11.5% tamaño). Motion 13.1.0 (breaking irrelevante para nosotros; trae `animateView`). LHCI y treosh: sin cambio. Biome 2.5.8 opcional; flag `.astro` sigue OFF + prettier-plugin-astro.
- **2 citas erróneas en `astro-css-tokens`**: Astro 6 nunca usó rolldown-vite (el bug era hoisting de Vite 8); el "#19802" real es de tailwindlabs. Corregir en B0. Node 20 se dropeó en Astro 6.1.0, no en 7.
- **furever-brand ya habla el contrato**: `tokens/web/{base,semantic,component}.json` es proyección string pre-emitida para SD v5 — se consume tal cual. Delta real: los **4 schemes** (dark mode + estados, 53 roles c/u) son objetos OKLCH sin proyección string → serializar con el patrón C-1 (~50 líneas zero-dep) a bloques CSS de override a nivel rol; default **light sin auto-switch** (G-UX-02), overrides al tier semántico. Menores: cubicBezier → envolver; tokens JS-vocabulary por import JSON; favicon desde `furever-iso-mono-*.svg`.
- **Contrato brand-system-skills 0.6.0**: spine de máquina fijo + gates auto-enforced copiados a cada repo emitido; web-stack-skills es el consumidor flagship (e2e verificado 2026-07-16, proyección `furever-web` ya registrada en `satellites/projections.md`). NO garantizado: schemes, component tier, fuentes bundleadas, y **ningún stamp de versión de contrato en el repo emitido** (el hop C-1→web ya se rompió 6 semanas una vez) → la skill #9 pinea contrato (0.6.0 + commit del tool-repo) con review-gate y corre `run-gates.mjs` como precondición.
- **Triggers de las 8**: 3 colisiones ALTAS (CSP motion↔security; inversión léxica "hash-based CSP" — 4 hermanas la nombran, la dueña no; "animated gradient background" sin hook en motion) + cluster medio C4–C10; 4 descriptions sin frontera; fixes de menor edición identificados. La auditoría es **prerequisito estructural** (la #9 agrega el eje "design tokens"/"brand" contra `astro-css-tokens` y contra los plugins brand-system en la misma superficie).

## Surfaces and models by phase
Chat es home base; Code es excursión (ir, hacer, volver con el resultado por referencia).

| Phase | Surface | Model role |
|---|---|---|
| A Cleanup & docs | Code ejecuta, chat adjudica (PR review) | top-reasoning (compactación + descriptions) |
| B0 Astro 7 wave | Code | top-reasoning (migración + edición de skills) |
| B1 brand-canon-ingest | chat (scope/decisiones) + Code (build) | top-reasoning |
| B2 Mechanical validation | Code | fast-tier (validate/package) + top-reasoning (misfires) |
| C Furever site | Code, paso a paso para Carlos | top-reasoning |
| D Integration playbook | chat | top-reasoning (síntesis) |
| E Ship v1 | chat + Code | fast-tier |

Nota para Carlos: Phase C sigue siendo la parte hands-on (terminal, git, deploy) — paso a paso, definiendo términos.

## Phase A — Cleanup & consolidation (docs)
Goal: repo legible, una sola fuente de verdad, superficie de triggers sana, listo para migrar.
- Commit de ESTE plan re-scopeado (rama `claude/`, PR, OK de Carlos).
- Compactar `RESIDENT.md` §11 EN SU LUGAR (una línea por turno hecho; lo load-bearing a §7/§8). One doc per repo — sin doc paralelo.
- Migrar al RESIDENT: roadmap v2 + principio Lego + "stack-governance NO es skill" + los hallazgos durables del reporte pre-sprint (Key inputs, arriba) → §8, y la tabla de re-pin objetivo → §3.
- **Descriptions fix-pass** (prerequisito estructural): aplicar los fixes identificados — "hash-based CSP" + frontera con redirects a `web-security-headers` (resuelve C1/C2/C5/C10), hook léxico de backgrounds/gradientes en `motion-system` (C3), fronteras en las 4 descriptions que no tienen, redirects a hermanas donde el headroom lo permita (ojo: signature-anim 1006/1024, perf-ci-gates 998/1024 — recortar antes de agregar). Producir tabla antes/después por skill.
- Crear `CLAUDE.md` del repo (gap de convención detectado): comandos, mapa, convención de skills, guardrails (validate/package, description ≤1024 sin `<`/`>` ni `: `).
- Verificar manifiestos (8 registradas, diferida fuera); README: ejemplo Lego concreto.

Gate A→B0: plan commiteado; RESIDENT compactado con lo durable migrado; descriptions parchadas sin colisiones ALTAS; CLAUDE.md en el repo.

## Phase B0 — Astro 7 migration wave (re-pin real)
Goal: la fundación al major vigente ANTES de validar y construir — no shippear v1 nacida vieja.
Decisión ya adjudicada (2026-08-17): migrar, no congelar en 6. Razones registradas: adapter 14 acoplado a ^7.2.0; la mina de Vite-8-hoisting hace Astro 6 frágil HOY; el switch-back de Tailwind a `@tailwindcss/vite` simplifica el stack; CSP sobrevive aditivo. Contra registrado: front-loads riesgo (~1 fase extra antes del rc).
- Re-pin lockstep por skill (tabla objetivo en RESIDENT §3 tras Phase A): astro 7.2.2 · @astrojs/cloudflare 14.2.1 · tailwindcss 4.3.3 + **switch-back a @tailwindcss/vite** (cerrar el review-gate; retirar la ruta postcss a nota histórica) · style-dictionary 5.5.1 (nunca 5.5.0) · biome 2.5.8 · rive 2.40.0 · sveltia 0.191.2 + smoke-test admin · motion 13.1.0 · gsap/@gsap/react/schema-dts/sitemap/lhci sin cambio.
- **Reemplazar astro-shield**: pieza SRI custom pequeña dentro de `web-security-headers`; retirar la dependencia abandonada.
- Corregir las 2 citas erróneas de `astro-css-tokens`; actualizar dato Firefox scroll-driven (~85.4%, watch Firefox 156 ~oct-2026); ajustar #1494 (workaround string solo typography, #1398 ya cerrado).
- Actualizar cada SKILL.md/reference afectada (pins, rutas de install, gotchas que el major invalide). Un commit por skill afectada; PR para review de Carlos.
- Verificación mínima: un scaffold Astro 7 de humo (no el sitio real) que compile con tokens + CSP + adapter 14 — prueba de que la fundación migrada se sostiene antes de B2.

Gate B0→B1: pins nuevos en RESIDENT §3; skills afectadas actualizadas y commiteadas; scaffold de humo compila.

## Phase B1 — Author brand-canon-ingest (#9)
Goal: la capacidad nueva, como skill del catálogo. Cadencia comprimida: el research ya está hecho (reporte pre-sprint, Bloques 3–4) → turns = Scope/decisiones (chat) → Build (Code) → validate/package/commit.
Alcance IN: consumir `tokens/web/*` tal cual como source de SD (delegando el pipeline a `astro-css-tokens`, sin duplicarlo); serializador de schemes patrón C-1 → CSS overrides a nivel rol (:root=lightA, variantes, [data-theme="dark"]), default light sin auto-switch; logos regla exact-file (los `-fc` prohibidos OFF-SYSTEM); iconos currentColor; fuentes woff2 → @font-face (+ nota de licencias owner-supplied); favicon/OG derivados (OG raster — G-IMG-03 prohíbe SVG); voz→copy (cargar canon.json + AMBOS keystones juntos); registro de la proyección en `satellites/projections.md`; data pointers (volátiles jamás congelados en el sitio → catalog.json/Supabase).
Condiciones de fábrica: (i) pin del contrato consumido (brand-system-skills 0.6.0 + commit del tool-repo) con review-gate; (ii) `run-gates.mjs` del repo de marca como precondición de la skill; (iii) description con fronteras filosas bilaterales — vs brand-canon-builder ("construir ≠ consumir") y vs `astro-css-tokens` ("design tokens" es trigger compartido: la ingest delega, no duplica).
**Precondición de Carlos (bloquea B1-build): ratificar el canon v2 de furever (Stage-10)** — gates 20/20 verdes, falta la firma.

Gate B1→B2: #9 autorada, validada, empaquetada, commiteada, registrada en `plugin.json`; canon ratificado; contrato pineado.

## Phase B2 — Mechanical validation (9 skills)
Goal: probar que el bundle instala y dispara — la "build final" original, ahora sobre 9.
- `quick_validate.py` + `package_skill.py` canónicos sobre las 9.
- `/plugin marketplace add` + `/plugin install web-stack@web-stack-skills`; ambos scopes.
- Test de triggering: cada skill dispara con sus frases objetivo y NO con las de una hermana — con foco en los ejes parchados en A (CSP, backgrounds) y el eje nuevo de #9 ("brand", "design tokens", vs plugins brand-system). Loguear misfires → fix → re-test.
- Taggear v1.0.0-rc.

Gate B2→C: install limpio en ambos scopes; triggering sin canibalización, incluido el eje de #9.

## Phase C — Furever reference site (the real integration test)
Goal: el sitio Furever vivo — integration test de composición + primera corrida real de #9 + activo comercial. Decisión adjudicada: el arquetipo ES Furever (resuelve el fork abierto del plan anterior).
- Componer el SUBCONJUNTO que el brief Furever pida (Lego): `brand-canon-ingest` + tokens + CSP + perf-gates + schema siempre; motion según brief; UN visual (webgl O rive) solo si el brief lo justifica; `cms-self-edit` (Sveltia) para contenido.
- Ejercer los seams DE VERDAD: ingest→tokens load order, schemes/dark, hidratación, CSP × JSON-LD × GSAP bundled, budget × motion, reduced-motion × a11y 0.95, CMS, deploy WSA.
- Si el brief jala forms o datos vivos → promover `forms-lead-system` / `data-layer` del backlog v2 EN ESE MOMENTO, bajo cadencia (no pre-construir).
- Puertas CI (LHCI + Biome) contra el build real.
- Capturar cada gotcha cross-cutting al momento (por referencia en el session log; detalle al playbook en D). Documentar el subconjunto compuesto ("composition recipe" Furever).
- **Gates de Carlos (no del stack, bloquean go-live, no el build):** GAP-006 banco de imagen (stock prohibido como sustituto), GAP-014 Aviso de Privacidad, doc de licencia de fuentes BTN.

Gate C→D: sitio vivo en Cloudflare pasando ambas puertas de CI; lecciones de campo capturadas. (Go-live público puede esperar los gates de Carlos sin bloquear D.)

## Phase D — Build stack-integration-playbook
Goal: promover la diferida a real, llenada desde Phase C.
- Cadencia de 5 turnos; contenido = lecciones de campo + seams ejercidos + cómo compone `brand-canon-ingest` con las demás (el seam de marca queda documentado aquí, no en la skill #9 — cada pieza en su especie).
- Composition recipes por arquetipo (Furever = la primera, real).
- Puntos de integración web ↔ resto del stack (Supabase, Actions, Cloudflare, Infisical, Workspace, pagos, analítica).
- Mover de `deferred/` a `skills/`, registrar en `plugin.json`.

Gate D→E: playbook con sustancia real, registrado. Desbloquea `client-discovery` (v2).

## Phase E — Ship v1
- Scope v1 = 9 skills + playbook, instalables, validadas, sitio Furever como prueba.
- Bump 1.0.0, tag release. README con el sitio como ejemplo + install verificado.
- RESIDENT actualizado (status: v1 shipped); archivar este plan (lo durable ya migró en A).

Done-criteria (externo, pre-comprometido): release 1.0.0 taggeado · install + triggering limpios de las 10 piezas (9 + playbook) · sitio Furever pasando ambas puertas de CI · playbook en `skills/` con sustancia de campo.

## v2 horizon — the Lego backlog
(Home canónico tras Phase A: RESIDENT §10. Aquí solo el puntero + lo comprometido.)
- `client-discovery` — único ítem comprometido/especificado; gate: v1 shipped + playbook (Phase D). Spec de 4 fases en RESIDENT tras la migración de Phase A.
- Backlog jalado por trigger: `data-layer`, `forms-lead-system` (ambos candidatos a que Furever los jale en C), a11y profundo, `i18n-system`, `media-optimization`, `edge-logic`, `analytics-measurement`, tier 3 especulativas. Invariantes v2: native-first, una herramienta por trabajo, review-gates en pins, descriptions sin solape, Lego.
- Drift honesto: ciclos de toque ~12–18 meses (majors de Astro, adapter). B0 es el primer ciclo real — presupuestar los siguientes.

## Open questions / forks
- Timing del go-live público de Furever vs gates de Carlos (GAP-006/GAP-014/licencias) — el build no espera; el go-live sí.
- `forms` / `data-layer`: decidir SOLO si el brief Furever los jala en C.
- Zonas huérfanas de triggers (view transitions, hover micro-interactions, Lottie): hoy caen a plugins de terceros; decidir en D si el playbook las mapea o si alguna amerita backlog v2.
- Nombre final de la skill v2 de intake (`client-discovery` vs alternativas): al autorarla.
- Resuelto (2026-08-17): Astro 7 sí (B0) · ingestión = skill nueva (B1) · sitio de referencia = Furever (C) · auditoría de descriptions = prerequisito estructural (A).

## Next actions
1. Code: commit de este plan en `claude/v1-sprint-rescope` + Phase A completa (compactación RESIDENT, migración de durables, descriptions fix-pass con tabla antes/después, CLAUDE.md, manifiestos, README) → PR → OK de Carlos.
2. Carlos (paralelo, no bloquea A): ratificar canon v2 furever Stage-10 — bloquea B1-build.
3. Chat: adjudicar la tabla antes/después de descriptions del PR; cerrar gate A→B0.

## Resume
Standing instruction: al abrir este plan en un chat fresco, leer el state block + front matter del RESIDENT, confirmar fase y next action con Carlos, ejecutar, loguear incrementalmente, y al cierre reescribir estado + session log. No reiniciar el plan.

## Session log
### 2026-08-17 — Re-scope del sprint desde el análisis pre-sprint
Hecho: análisis profundo pre-sprint vía Code (11 agentes, 24 claims re-verificados adversarialmente: 23 confirmados, 1 corregido en atribución) — estado del repo, re-pin audit completo, inventario furever-brand, contrato brand-system-skills. Los 3 repos clonados/sincronizados en ~/proyectos/. Plan re-scopeado a 7 fases.
Decidido: (1) migrar a Astro 7 antes de validar (B0) — adapter 14 acoplado, mina Vite-8 en Astro 6, switch-back Tailwind limpio; (2) ingestión de marca = skill nueva `brand-canon-ingest` (#9, B1) con pin de contrato 0.6.0 + run-gates como precondición — no extensión de astro-css-tokens (una herramienta por trabajo) ni recipe del playbook (bloquearía la capacidad tras Phase D); (3) sitio de referencia = Furever (C) — integration test + primera corrida de #9 + activo; (4) descriptions fix-pass promovida a prerequisito estructural (3 colisiones ALTAS + eje nuevo de #9); (5) astro-shield se reemplaza DENTRO de B0 (abandonado, peer ^4).
Dead-ends: raw.githubusercontent y la página renderizada de GitHub sirven copias stale del repo — leer estado vía clon local o Contents API, nunca el render. El .7z de Downloads era idéntico al HEAD (solo CRLF) — el clon es canónico.
Siguiente: commit de este plan + Phase A (Code); Carlos ratifica canon Stage-10 en paralelo.

### 2026-06-18 — cms-self-edit (#8) cerrada; plan re-scopeado a 2 horizontes
Hecho: autorada y commiteada la 8ª skill `cms-self-edit` (commit `90110a9e`; RESIDENT `d7da1b25`). Gate de arranque LIBRE.
Decidido: meta a 2 horizontes (H1 ship v1, H2 cliente-facing); `client-discovery` añadida a v2 como único ítem comprometido (4 fases; depende del playbook). Auditoría de factibilidad consolidada DENTRO de esa skill.
Siguiente (superseded por el re-scope 2026-08-17): Phase A del scope anterior.

## Limitations & Out-of-Scope
- NO pre-construye skills v2 — las jala un proyecto real (incluido el propio sitio Furever en C).
- El harness sintético de stress-tests queda FUERA — el sitio Furever es el integration test.
- La tesis comercial / GTM vive en notas privadas de Carlos, no en este doc de repo público.
- Los gates de contenido de Carlos (imágenes, Aviso de Privacidad, licencias) bloquean go-live, no el sprint técnico.

## References
- [RESIDENT.md](./RESIDENT.md): fuente de verdad de hechos durables. Apuntar, no duplicar.
- [plugin.json](./.claude-plugin/plugin.json) y [marketplace.json](./.claude-plugin/marketplace.json): manifiestos.
- Las `skills/<nombre>/SKILL.md`: verdicto, pins, gotchas por skill.
- `~/proyectos/furever-brand`: repo de marca consumido (contrato brand-system-skills 0.6.0; proyección registrada en `satellites/projections.md`).
- `~/proyectos/brand-system-skills`: tool-repo del contrato (pin de referencia para el review-gate de #9).

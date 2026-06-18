---
name: web-stack-v1-finalization
description: "Execution plan Claude works from to take ccediland/web-stack-skills from cms-self-edit complete to v1 shipped — cleanup, mechanical validation, a reference-site composition test, the integration playbook, and release — plus the v2 Lego roadmap. Use when resuming this phase in a fresh chat after cms-self-edit (#8) is authored, validated, and committed."
title: "web-stack-skills — v1 Finalization & v2 Roadmap"
summary: "Mortal execution plan for the post-cms-self-edit phase. Five gated phases (A cleanup, B mechanical validation, C reference-site composition test, D integration playbook, E ship v1) plus the open v2 backlog. Points at RESIDENT.md for standing facts; archived at v1 ship."
last_updated: 2026-06-17
applies_to: "ccediland/web-stack-skills · post-7-skills + cms-self-edit · pre-v1-ship"
status: "NOT STARTED — gated on cms-self-edit (#8) authored + validated + committed + registered in plugin.json"
phase: "A (Cleanup) is first"
home_base: "chat (claude.ai); excursion to Claude Code for phases B and C"
next_action: "Confirm cms-self-edit (#8) is done; then Phase A, step 1"
resident: "./RESIDENT.md — canonical for all standing facts (skill verdicts, pins, decisions, state). Point, never duplicate."
---

# web-stack-skills — v1 Finalization & v2 Roadmap

> Plan de ejecución para la fase que arranca cuando `cms-self-edit` (#8) quede lista: limpiar el repo, validar que el bundle instala y compone, construir el `stack-integration-playbook` desde un sitio real, y shippear v1 — más el roadmap v2 abierto. Es un doc mortal (estado de sesión); los hechos durables viven en `RESIDENT.md`.

## TL;DR
- Objetivo acotado: llevar el repo de "8 skills redactadas, cero sitios" a "v1 shippeada con un sitio de referencia vivo como prueba". Secuencia con compuertas: A limpieza → B validación mecánica → C sitio de referencia (integration test real) → D integration playbook → E ship v1.
- Caveat (se repite a propósito): este plan es MORTAL. Las skills, pins y decisiones durables viven en `RESIDENT.md`. Phase A migra el roadmap v2 al RESIDENT. Al shippear v1, este plan se archiva — no se vuelve un segundo doc de conocimiento.
- Caveat: el sitio de referencia ES el integration test. NO se construye un harness sintético de stress-tests con logs en muchos CLAUDE.md; las lecciones de campo del build real son las que llenan el playbook.
- Principio Lego: el bundle es un catálogo de partes. Cada sitio compone un SUBCONJUNTO. Ningún sitio lleva todas las skills. Las skills nuevas amplían el alcance del catálogo, no la carga de cada sitio.
- Las cuatro innegociables (performance, SEO/AEO, visuales, ciberseguridad) YA quedan cubiertas por el core de v1. v2 amplía alcance funcional, no el piso de calidad.

## When to use this document
Úsalo en un chat fresco DESPUÉS de que `cms-self-edit` (#8) esté autorada, validada, commiteada y registrada en `plugin.json`. Si #8 no está lista, este plan no arranca — termínala primero bajo la cadencia del RESIDENT. Al abrirlo: lee el state block del front matter y el front matter + §3/§9/§10 del RESIDENT, confirma la compuerta, confirma la next action con Carlos, ejecuta la fase actual, y al cerrar añade al session log sin reiniciar el plan.

## Objective
v1 = las 7 skills fundación/visuales + `cms-self-edit` + `stack-integration-playbook`, todas instalables, con install y triggering validados en Claude Code, y UN sitio de referencia vivo en Cloudflare pasando ambas puertas de CI (LHCI + Biome) como prueba de composición. Done-criteria explícito en Phase E.

## Starting state (gate to start)
Precondición dura antes de Phase A:
- `cms-self-edit` (#8) autorada por la cadencia de 5 turnos (incluido su turno de selección de herramienta), validada, commiteada, y registrada en `plugin.json`.
- El CMS elegido quedó registrado en `stack-canon-argos` (era un gap del stack-canon).
- Repo en el estado del RESIDENT: 7 skills fundación/visuales completas + #8 + `stack-integration-playbook` aún como skeleton en `deferred/`.

Si algo de esto falta, no se inicia el plan.

## This document's lifespan
Es un execution plan (species 1 de ai-roadmap): estado mutable de sesión, no contexto permanente. `RESIDENT.md` es el doc vivo perpetuo y la única fuente de verdad de hechos durables — verdictos de skills, pins, decisiones, estado. Este plan los CITA por sección, nunca los copia (copias driftan y una miente al mes). Phase A migra los pedazos durables de este plan (roadmap v2, principio Lego, la decisión "stack-governance no es skill") al RESIDENT, para que al archivar este plan en Phase E no se pierda nada.

## The Lego principle
El bundle es un catálogo, no un sistema fijo. Cada sitio compone solo las skills que su brief necesita. Reglas:
- Ningún sitio usa todas las skills. Un brochure corporativo no lleva data-layer; un menú de restaurante sí. La capa visual entra UNA vez, cuando el brief la justifica, nunca por default.
- Una skill nueva amplía el alcance del catálogo, no el payload de cada sitio.
- El `stack-integration-playbook` es la columna que hace posible la composición arbitraria: documenta cómo conectan las partes entre sí y con el resto del stack. Por eso su prioridad de llenado sube en cuanto haya sustancia de campo.

## v1 coverage of the non-negotiables
Lo que Carlos quiere en todo sitio, y la skill de v1 que ya lo cubre. v1 cierra el piso; v2 es alcance.

| Innegociable | Skill de v1 que lo cubre |
|---|---|
| Performance | `perf-ci-gates` (+ disciplina de budget de `motion-system`) |
| SEO / AEO | `seo-aeo-schema` |
| Visuales | `motion-system` + `webgl-atmosfera` + `signature-anim` (uno, cuando el brief lo pida) |
| Ciberseguridad | `web-security-headers` |

## Surfaces and models by phase
Chat es home base; Code es excursión (ir, hacer, volver con el resultado por referencia). Modelo nombrado por rol, no por versión.

| Phase | Surface | Model role |
|---|---|---|
| A Cleanup | chat | top-reasoning (destilación + auditoría de solape) |
| B Mechanical validation | Claude Code | fast-tier (validate/package) + top-reasoning (fixes de description) |
| C Reference site | Claude Code | top-reasoning (build + seams); paso-a-paso para Carlos |
| D Integration playbook | chat | top-reasoning (síntesis de lecciones) |
| E Ship v1 | chat + Composio/Code | fast-tier (mecánica de release) |

Nota para Carlos: Phase C es la parte hands-on de dev (terminal, Claude Code, git, deploy en Cloudflare) — la curva de aprendizaje. Es esperable que sea lo más lento; el chat que la ejecute debe ir paso a paso, definiendo términos, sin asumir procedimiento.

## Phase A — Cleanup & consolidation
Goal: repo legible, una sola fuente de verdad, listo para validar.
- Compactar `RESIDENT.md` §11 (session log) EN SU LUGAR: destilar los turnos hechos a una línea cada uno, preservando lo load-bearing en §7 (decisiones) y §8 (descubrimientos). No abrir un doc paralelo — one doc per repo.
- Migrar al RESIDENT (§10 Roadmap / §3) como hogar canónico: el roadmap v2 de abajo, el principio Lego, y la decisión "stack-governance NO es skill".
- Verificar `plugin.json` + `marketplace.json`: `cms-self-edit` registrada como 8ª skill; la diferida sigue fuera; subir el campo de versión hacia release.
- README: agregar un ejemplo concreto de qué compone un sitio (antes vs después) + enunciar el principio Lego. Opcional: nombrar el nicho como posicionamiento (no es secreto) si Carlos lo decide.
- Pase de higiene: cada skill bajo `skills/<nombre>/` y registrada; diferida fuera de `skills/`; auditar que las `description` de las 8 sean filosas y SIN solape de triggers (la superficie más propensa a fallar — el revisor externo la marcó). Producir la lista de cualquier colisión de triggers entre skills.

Gate A→B: RESIDENT compactado, roadmap v2 migrado al RESIDENT, 8 skills registradas, descriptions auditadas sin solape.

## Phase B — Mechanical validation
Goal: probar que el bundle instala y dispara. Esta es la "build final" que §6 reservó para Claude Code — la primera vez que se abre.
- Abrir Claude Code (ya permitido — todas las skills literal). Entorno: WSL2/Ubuntu, git/gh como `ccediland`.
- Correr los scripts canónicos `quick_validate.py` + `package_skill.py` de skill-creator sobre cada skill (los reales, no la reproducción por Composio usada al autorar). Arreglar fallos de validación.
- `/plugin marketplace add ccediland/web-stack-skills` → `/plugin install web-stack@web-stack-skills`. Verificar install a scope de proyecto y personal.
- Test de triggering (el stress-test real de la superficie de descriptions): por cada skill, confirmar que dispara con sus frases objetivo y NO dispara con las de una hermana. Loguear misfires, arreglar descriptions, re-validar, re-testear.
- Taggear un pre-release (v1.0.0-rc).

Gate B→C: install limpio en ambos scopes; cada skill dispara correcto sin canibalización entre hermanas.

## Phase C — Reference site (the real integration test)
Goal: probar que las skills componen en un sitio real; cosechar lecciones de campo; el sitio dobla como activo propio de Carlos / prueba de venta.
- Elegir el arquetipo. Recomendado: corporativo-de-imagen o consultoría-AI (sweet-spot + se vuelve el sitio de su propia empresa / portafolio).
- Scaffold del sitio Astro 6 + Cloudflare con Claude Code, componiendo el SUBCONJUNTO que el brief pide (Lego): tokens + CSP + perf-gates + schema siempre; motion según haga falta; UN visual (webgl O rive) solo si el brief lo justifica; `cms-self-edit` para contenido.
- Ejercer los seams DE VERDAD (no en papel): load order, hidratación de islas, CSP × JSON-LD inline × scripts GSAP bundled, budget de perf × peso de motion, reduced-motion × puerta a11y (0.95), flujo de datos del CMS, deploy en Cloudflare.
- Correr las puertas de CI (LHCI + Biome) contra el build real: ¿la capa visual mantiene los CWV dentro del budget?
- Capturar cada gotcha cross-cutting al momento. Estas son las lecciones de campo irremplazables. Registrarlas por referencia en el session log de este plan; el detalle completo va al playbook en Phase D.
- Documentar el subconjunto compuesto y por qué (se vuelve una "composition recipe" del playbook para ese arquetipo).

Gate C→D: sitio de referencia vivo en Cloudflare, pasando ambas puertas de CI, con la lista de lecciones de campo capturada.

## Phase D — Build stack-integration-playbook
Goal: promover la skill diferida a real, llenada desde Phase C (no desde teoría).
- Correrla por la cadencia de 5 turnos (hoy no la pasa). Contenido manejado por las lecciones de Phase C.
- Llenar: cómo componen las skills en un sitio real (load order, hidratación, seams); gotchas cross-cutting; puntos de integración web ↔ resto del stack (Supabase, GitHub Actions, Cloudflare, Infisical, Google Workspace, comms, pagos, analítica — su scope de skeleton).
- Agregar composition recipes por arquetipo (los subconjuntos Lego): corporativo-imagen / landing-premium / restaurante+menú / storytelling / consultoría-AI → qué skills compone cada uno, cuáles salta.
- Mover de `deferred/` a `skills/`, registrar en `plugin.json`.

Gate D→E: playbook pasa la cadencia, tiene sustancia de un sitio real, registrado en el plugin.

## Phase E — Ship v1
Goal: v1 liberada.
- Scope v1 = 7 skills fundación/visuales + `cms-self-edit` + `stack-integration-playbook`, todas instalables, install + triggering validados, un sitio de referencia vivo como prueba.
- Bump a 1.0.0, taggear release.
- README: sitio de referencia como el ejemplo concreto; principio Lego enunciado; instrucciones de install verificadas.
- Actualizar RESIDENT (status: v1 shippeada) y archivar este execution plan (sus partes durables ya están en el RESIDENT desde Phase A).

Done-criteria de v1 (completion criterion externo y pre-comprometido — esto cierra v1, no un estado de "se siente lista"):
- release taggeado 1.0.0, y
- install + triggering limpios en Claude Code, y
- un sitio de referencia vivo pasando ambas puertas de CI, y
- `stack-integration-playbook` en `skills/` con sustancia de campo.

## v2 horizon — the Lego backlog
v1 = core probado que cubre los casos sweet-spot y las cuatro innegociables. v2 = ampliar el ALCANCE funcional agregando skills JALADAS por proyectos reales, cada una manteniendo los invariantes. Nada de roadmap por brainstorm. (Home canónico de esta sección tras Phase A: RESIDENT §10.)

Candidate backlog con build-trigger (la señal de proyecto real que la promueve de backlog a autoría):

| Skill candidata | Tier | Build-trigger |
|---|---|---|
| `data-layer` (datos externos, filtros/búsqueda/interactividad, live data, lógica de negocio) | 1 | un proyecto pide menú/catálogo/datos dinámicos — sesgar a build-time, no live |
| `forms-lead-system` (o pick en stack-canon + receta en playbook) | 1 | un proyecto necesita captura/leads/contacto |
| a11y profundo (más allá del piso 0.95 de perf-ci-gates) | 2 | un cliente exige WCAG AA formal |
| `i18n-system` | 2 | un proyecto multi-idioma real |
| `media-optimization` | 2 | un sitio image/video-heavy lo exige |
| `edge-logic` (Workers: A/B, redirects, geo, feature flags) | 2 | un proyecto necesita lógica en el edge |
| `analytics-measurement` (privacy-friendly + eventos) | 2 | un cliente pide medición real |
| view-transitions / conversion-patterns / content-modeling / speculation-rules / auth-simple / legal-compliance / component-scaffolding / visual-regression-ci | 3 | especulativas — solo si un proyecto las jala |
| `stack-governance` | ✗ | NO es skill — la cadencia ya hace review-gates; vive como sección de RESIDENT |

Invariantes que toda skill v2 debe mantener (para que el bundle siga coherente y de bajo mantenimiento):
- native-first: antes de agregar una capa, declarar qué opción nativa se descartó y por qué.
- una herramienta por trabajo, sin solape.
- review-gate en cada pin duro y cada workaround por issue-number (la disciplina de drift; solo + sin auto-update = half-life corto → presupuestar mantenimiento activo, no asumir estabilidad).
- description filosa y sin solape (la superficie de discoverability).
- buy/configure sobre custom-build; custom = excepción documentada.
- regla Lego: la skill amplía el alcance del catálogo, no el payload de cada sitio.

Drift / setup-and-forget (postura honesta, ya en la revisión externa del repo): el setup-and-forget puro NO aplica a los sitios construidos sobre esto. Esperar ciclos de toque de ~12–18 meses (majors de Astro, cambios del adapter de Cloudflare). Mitigaciones: mantener los sitios tan estáticos como el brief permita; sesgar `data-layer` a build-time; cada pin con review-gate; el playbook documenta los seams frágiles. v2 mantiene esta disciplina — no elimina el mantenimiento, lo acota.

## Open questions / forks
Las pocas decisiones reales; resolver al llegar al punto, no antes.
- Audiencia de la consolidación del RESIDENT: compactar-en-su-lugar (default, una fuente de verdad) vs además un `ARCHITECTURE.md`/README expandido para visitantes del repo. Default: compactar en su lugar; agregar doc público solo si quieres legibilidad para extraños.
- Arquetipo del sitio de referencia: corporativo-imagen vs consultoría-AI. Recomendado: el que doble como tu activo real de empresa.
- `forms`: skill propia vs pick en stack-canon + receta en playbook. Decidir cuando data-layer/forms se jale por primera vez.
- CMS de `cms-self-edit`: lockeado en su propio turno de selección (pre-este-plan). Este plan hereda lo elegido; solo confirmar que quedó registrado en stack-canon.

## Next actions
Para el chat fresco que abra este plan, en orden:
1. Confirmar que `cms-self-edit` (#8) está autorada, validada, commiteada y registrada en `plugin.json`. Si no, este plan no arranca — terminarla primero.
2. Leer front matter + §3/§9/§10 del RESIDENT para confirmar el estado actual.
3. Iniciar Phase A, step 1 (compactar §11 del RESIDENT en su lugar).

## Resume
Standing instruction: al abrir este plan en un chat fresco, leer el state block del front matter + front matter del RESIDENT, confirmar la compuerta de arranque (cms-self-edit done), confirmar la next action con Carlos, y ejecutar la fase actual. Loguear incrementalmente; al cierre reescribir el estado y añadir la entrada al session log. No reiniciar el plan ni re-derivar lo que el log ya resolvió.

## Session log
(Vacío — añadir una entrada destilada por sesión: qué se hizo, qué se decidió, dead-ends, siguiente. Mantener la entrada más reciente completa; comprimir las viejas a decisiones y punteros.)

## Limitations & Out-of-Scope
- NO cubre autorar #6 `webgl-atmosfera`, #7 `signature-anim`, ni `cms-self-edit` — esas terminan bajo la cadencia del RESIDENT ANTES de que arranque este plan.
- NO construye todas las skills v2 — esas las jala un proyecto real, no se calendarizan aquí.
- El harness sintético de stress-tests queda deliberadamente FUERA — el sitio de referencia es el integration test.
- La tesis comercial / GTM se mantiene en las notas privadas de Carlos, no en este doc de repo público.

## References
- [RESIDENT.md](./RESIDENT.md): fuente de verdad canónica de hechos durables (verdictos de skills, pins, decisiones, estado, roadmap). Apuntar, no duplicar.
- [plugin.json](./.claude-plugin/plugin.json) y [marketplace.json](./.claude-plugin/marketplace.json): manifiesto del plugin y del marketplace.
- Las 8 `skills/<nombre>/SKILL.md`: verdicto, pins, gotchas, outline de cada skill.

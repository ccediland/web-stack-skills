---
name: web-stack-full-catalog
description: "Execution plan Claude works from to take ccediland/web-stack-skills from 9-skills-authored to the complete catalog shipped — mechanical validation, the Furever integration fixture, the integration playbook, five capability waves covering the entire backlog (data, forms, i18n, media, a11y, edge, analytics, full tier 3), and the client-facing layer culminating in client-discovery and v2.0.0. Use when resuming this work in a fresh chat."
title: "web-stack-skills — Full Catalog Roadmap (v1.0.0 → v2.0.0)"
summary: "Mortal execution plan, re-scoped 2026-08-18 by owner directive: the full backlog is BUILT, not declared — pull-by-project retired as a deliberate deviation; Lego survives at composition time. Furever is a permanent integration fixture (temp Workers deploys at gates, never a final build). Sequence: B2 validation → C fixture → D playbook (ship 1.0.0) → W1–W4 capability waves (1.1–1.4) → W5 client layer (2.0.0). Points at RESIDENT.md for standing facts; archived at v2 ship."
last_updated: 2026-08-18
applies_to: "ccediland/web-stack-skills · post-B1 (9 skills authored, Astro 7 baseline) · consumes ccediland/furever-brand as test fixture (contract 0.6.0)"
status: "IN PROGRESS — Phase B2 EXECUTED and merged: plan replaced in-repo, descriptions unfrozen to Astro 7 (7 skills + both manifests), 9/9 validate+package, real install both scopes (local-path marketplace), triggering 30/30 blind with zero misfires, Sveltia 0.191.2 smoke PASS, manifests at 1.0.0-rc, tag pushed."
phase: "C (Furever integration fixture) — ready to start"
home_base: "chat (claude.ai) for adjudication only; all execution and analysis in Claude Code (Fable 5)"
next_action: "Code: Phase C — compose the fixture subset in furever-web, decide brand-repo access (open fork: sibling clone / submodule / CI checkout), exercise the seams, CI gates, TEMP deploy to a Workers subdomain (owner names the domain if not workers.dev)"
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
- Acceso del sitio al repo de marca (clone hermano / submodule / checkout CI) — decidir en C con evidencia.
- forms-lead-system: skill vs pick+recipe — Run A de W1.
- Veredictos de hogar de W4 — Run A de W4 (leans arriba).
- Nombre final de client-discovery — Run A de W5.
- Subdominio de prueba para deploys del fixture (workers.dev default vs subdominio propio del dueño) — decidir en C; irreversible=no, pero el dueño nombra el dominio.
- Resueltos (2026-08-18): catálogo completo se construye (owner) · Furever = fixture permanente, deploys temporales, sin go-live · ratificación Stage-10 = gobernanza post-sprint, fuera del plan.

## Next actions
1. Chat: emitir el prompt de Phase C (fixture Furever en `furever-web`); adjudicar ahí el fork de acceso al repo de marca y el subdominio de deploy temporal si Code lo escala.
2. Code (Phase C): componer el subconjunto del fixture, ejercitar los seams, CI gates, deploy temporal Workers → reporte destilado al chat.

## Resume
Standing instruction: en chat fresco — leer state block + RESIDENT (front matter, §3, §8, §9, §10), confirmar fase y next action con Carlos, ejecutar, loguear incrementalmente, cerrar con estado + session log + hand-off. No reiniciar el plan.

## Session log
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

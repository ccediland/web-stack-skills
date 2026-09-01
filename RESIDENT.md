---
name: web-stack-skills-resident
title: web-stack-skills — RESIDENT
description: Marketplace de plugins de Claude Code que codifica un stack web premium de alto rendimiento (Astro + Cloudflare Workers) en 19 skills. Cómo es el catálogo hoy, su stack, sus reglas operativas, el índice de decisiones y hacia dónde va.
last_updated: 2026-09-01
status: catálogo completo v2.0.0 — modo mantenimiento
supersede: ninguno
---

# web-stack-skills — RESIDENT

Documento vivo: fuente de verdad única de hechos durables sobre este catálogo. La narrativa de cómo
se construyó —sesión por sesión, con fechas— vive en `BITACORA.md`; aquí solo queda lo que sigue
siendo verdad sin que nadie lo toque.

## 1. Qué es

Un **marketplace de plugins de Claude Code**, público y genérico (MIT), que entrega **19 skills**
que codifican un stack web premium de alto rendimiento: 17 de capacidad + `client-discovery` (la capa
cliente: del material del cliente al brief con veredictos de factibilidad) +
`stack-integration-playbook`, la autoridad de composición (las recipes de capacidad y de arquetipo).
Reusable en cualquier proyecto: un proyecto origen fue el primer consumidor, **no** el alcance. Cero
contenido específico de proyecto, cero marca, cero secretos.

Meta: parándote en cualquier proyecto nuevo, levantar la misma arquitectura sin volver a descubrir
los filos.

**Principio Lego** (regla permanente): el bundle es un catálogo, no un sistema fijo. Cada sitio
compone solo el subconjunto que su brief necesita; ningún sitio lleva todas las skills; la capa
visual entra UNA vez, cuando el brief la justifica, nunca por default. Una skill nueva amplía el
catálogo, no el payload de cada sitio.

## 2. El stack

Astro 7 · Cloudflare Workers Static Assets · Tailwind v4 + Style Dictionary · GSAP + CSS
scroll-driven + Motion · OGL/raw WebGL2 · Rive · schema-dts + `@astrojs/sitemap` + `llms.txt` · CSP
nativo · Lighthouse CI + Biome.

## 3. Las 19 skills

El roster completo con su qué-hace en una línea vive en `README.md` (tabla pública). El veredicto
extendido de las skills que llevan disciplina de selección o composición propia (cms-self-edit,
brand-canon-ingest, data-layer, i18n-system, media-optimization, edge-logic, auth-simple,
visual-regression-ci, conversion-patterns, client-discovery, a11y-deep, el playbook y sus recipes de
arquetipo) más las dos tablas de pins viven en `docs/skill-verdicts.md`. El detalle canónico completo
de cada una —veredicto, pins propios, gotchas, outline— vive en su propio
`plugin/skills/<nombre>/SKILL.md`.

## 4. Arquitectura del repo

`plugin/` es el ÚNICO subdirectorio que viaja al installer — no existe ignore mechanism en plugins,
el source dir se copia completo, así que el subdirectorio ES el filtro. Los docs del repo
(`RESIDENT.md`, `CLAUDE.md`, `BITACORA.md`, `README.md`, `archive/`) viven fuera. Cada skill vive en
`plugin/skills/<nombre>/` y debe registrarse en el array `"skills"` de `plugin.json` — el
auto-discovery solo no basta. `deferred/` (fuera del plugin) es para skills sin sustancia todavía.
Mapa completo de archivos: `MAPA.md`.

## 5. Cadencia de autoría (5 turnos por skill, ~1 skill por chat)

| Turno | Qué |
|---|---|
| 1 | Entendimiento / scoping — delimitar cobertura, proponer estructura del bundle |
| 2 | Pre-research — skill `pre-research` (web_search + pre-brief) |
| 3 | Research — Research mode + Context7; re-verificar versiones |
| 4 | Cierre / decisiones / preguntas |
| 5 | Build — autoría del bundle → `quick_validate.py` → `package_skill.py` → commit; cierre: RESIDENT + BITACORA + hand-off |

## 6. Reglas operativas

- **Un plugin** (`web-stack`) agrupa las 19 skills. Instalación:
  `/plugin marketplace add ccediland/web-stack-skills` luego `/plugin install web-stack@web-stack-skills`.
- Gobierno: **skill-author** = autoridad de arquitectura; **skill-creator** (`quick_validate.py` +
  `package_skill.py`) = validar/empacar; **Context7** + docs oficiales = contenido actual.
- Naming: kebab-case, ≤64, sin la palabra "claude". Description ≤1024, sin `<`/`>`, sin
  dos-puntos-espacio a media cadena (rompe YAML).
- Cada pin duro y cada workaround por issue-number lleva review-gate (§3 de `skill-verdicts.md`); no
  bumpear pins casualmente — los bumps van por fase de re-pin o por review-gate disparado.
- `SKILL.md` = veredicto + receta; `references/` = configs/plantillas/gotchas (progressive
  disclosure).

## 7. Decisiones (índice — el registro completo vive en `BITACORA.md`)

- `D-001` — repo público, genérico, MIT; un marketplace con un plugin agrupando el catálogo.
- `D-002` — skill sin sustancia se difiere como skeleton, fuera del instalable.
- `D-003` — layout `skills/<nombre>/` + registro explícito en `plugin.json`.
- `D-004` — el activo durable es el MÉTODO (RESIDENT + cadencia + playbook), no las skills.
- `D-005` — disciplina de perecederos: cada pin/workaround lleva review-gate.
- `D-006` — la capa visual es doble filo: una por sitio, solo si el brief la justifica.
- `D-007` — sin smoke-test temprano de install (decisión de Carlos).
- `D-008` — la tesis comercial/GTM vive fuera del repo público.
- `D-009` — auth de cms-self-edit: Sveltia default salvo rechazo de GitHub por el cliente.

Cada una con su fecha y su razonamiento completo en `BITACORA.md`.

## 8. Calendario de drift

El catálogo en modo mantenimiento vigila 18 disparadores de obsolescencia (versiones, pricing,
reformas legales) que, al moverse, exigen re-verificar una skill o recipe contra su fuente primaria.
Índice completo con dueña y acción: `docs/calendario-drift.md`. Los dos más volátiles hoy: madurez y
pricing de **Worker-level Access** (auth-simple, primer burn real hecho el 2026-08-18) y la
**reforma LFPDPPP** en el Congreso mexicano (~sept-2026, legal recipe del playbook).

## Rumbo

**Objetivo vigente:** estable; sin rumbo activo de construcción. El catálogo está completo
(v2.0.0, cierre 2026-08-18) y en modo mantenimiento — el trabajo entra por el calendario de drift
(§8) y por burns de skills nuevas contra el fixture permanente (`furever-web`, rama `claude/c-fixture`,
que JAMÁS se mergea a su main = producción).

**Lo que sigue** (el tablero vivo es `gh issue list`; hoy está vacío — el trabajo entra por disparo
del calendario de drift, no por cola de tickets):

- Sin frentes abiertos hoy.

**Despensa** (mencionado, no comprometido):

- Decidir si las upstream suggestions `S-1`..`S-4` (`upstream-suggestions.md`) se formalizan como
  issues en `brand-system-skills`.
- `client-portal` (cuentas de cliente con datos por usuario) como recipe propia del playbook — hoy es
  frontera documentada, no comprometida; solo si un proyecto real la necesita.
- Flip a Resend en `forms-lead-recipe.md` si aparece necesidad de destinatarios arbitrarios (hoy
  Cloudflare Email Service basta con verified-destination).

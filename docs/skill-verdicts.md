---
title: web-stack-skills — veredictos extendidos y pins
summary: Detalle técnico por skill que no cabe en el presupuesto de RESIDENT.md — el veredicto largo de las skills #8 a #19 (las que llevan disciplina de composición o selección por mérito) y las dos tablas de pins. Referencia atemporal, no narrativa: se actualiza cuando el veredicto cambia, no por sesión.
last_updated: 2026-09-01
applies_to: web-stack-skills
---

# web-stack-skills — veredictos extendidos y pins

El resumen de una línea por skill vive en `README.md` (tabla pública). El detalle canónico completo
de cada una —veredicto, pins propios, gotchas, outline— vive en su `plugin/skills/<nombre>/SKILL.md`.
Este archivo es el punto medio: el veredicto extendido de las skills cuya selección o composición no
cabe en una línea, más las tablas de pins que el RESIDENT no puede cargar sin romper su presupuesto.

## Veredictos extendidos

**`stack-integration-playbook`** (skill de COMPOSICIÓN, nunca de capacidades individuales — cada una
redirige a su dueña). El orden canónico probado (ingest → tokens → security → gates → seo → motion →
UN visual → cms) es una cadena de dependencias, no un ritual: omitir pasos es Lego, reordenarlos no.
Los seams entre skills son donde los builds compuestos fallan, y los dos peores modos de fallo son
SILENCIOSOS (gate vacuo, shader por CPU solo en runners). Bundle: SKILL.md (orden + reglas de omisión
+ disciplina de verificación) + 3 refs — `seams-and-gotchas` (9 seams + secundarios, cada uno con su
modo de fallo probado), `stack-integration-map` (puntos de contacto web↔stack: Supabase, Actions,
Cloudflare, Infisical, Workspace), `recipes` (recipe #1 brand-heavy probada por el fixture +
skeleton-contrato para arquetipos).

**`cms-self-edit`.** Disciplina para dar a un cliente NO técnico la capacidad de auto-editar
contenido, sin lock-in tipo Webflow/Framer. Ganador **Sveltia** — CMS git-based client-side puro,
`public/admin/index.html` sin backend, contenido en Markdown/YAML, OAuth GitHub vía el Worker
`sveltia-cms-auth`, media en R2, i18n first-class (DeepL deshabilitado en Sveltia — traducción vía
Google Cloud Translation/Gemini/Mistral). Escalera de 3 etapas: Sveltia (default) → Pages CMS (el
cliente jamás ve GitHub) → Directus (contenido relacional/multi-canal, ~$200/mes). Descartados:
Keystatic (roto en Astro 6, sin i18n), EmDash (dev-preview), Tina (pesado), Decap (abandonado, Sveltia
es su sucesor), Sanity/headless-DB self-host (overkill salvo Etapa 3).

**`brand-canon-ingest`.** Disciplina de CONSUMO de un repo de marca (contrato `brand-system-skills`
0.6.0) hacia un sitio del stack: el repo de marca es la única fuente de verdad, el sitio es una
PROYECCIÓN registrada. Precondición no negociable: `run-gates.mjs` del repo de marca en verde.
Tokens copiados verbatim, schemes serializados por un patrón C-1 vendorizado (~50 líneas zero-dep),
`:root` = default light PRIMERO (empate de especificidad). Assets exact-file para logos, iconos
currentColor. Gobernanza: registrar la proyección en `satellites/projections.md` del repo de marca;
protocolo root→gates→re-ingest, jamás parchar site-side.

**`data-layer`.** Datos externos/de negocio con sesgo BUILD-TIME native-first: escalera de 4 tiers
donde cada salto se gana con el brief — tier 0 `file()` sobre seed → tier 1 (default) custom loader
leyendo Supabase en build → tier 2 client-side (llave expuesta por diseño con RLS de frontera) → tier
3 live collections (fuera del default, decisión de arquitectura). Rebuild nativo: Supabase Database
Webhook → Workers Builds Deploy Hook. Un solo zod schema como contrato.

**`i18n-system`.** Multi-idioma con Astro CORE i18n, cero dependencias. Default locale en raíz,
strings en un diccionario tipado zero-dep, colecciones en `multiple_folders` (layout que Astro y
Sveltia comparten), traducciones enlazadas por `translationKey`. Hreflang emitido por `seo-aeo-schema`
(el mapa es de esta skill, la superficie que lo emite es de la otra — frontera bilateral). Sin
detección/redirect por default.

**`media-optimization`.** Skill delgada de producción de media. `imageService: 'compile'` explícito
(el default del adapter cambió a `cloudflare-binding` en v13.0.0, cuyo free tier revienta cerrado a
5k uniques/mes). Responsive vía `layout: 'constrained'` + `responsiveStyles: true`; `priority` solo en
la imagen LCP. Video en tiers forzados por plataforma (mp4 en `public/` → R2 custom domain → Media
Transformations → Stream).

**`edge-logic`.** Skill delgada de lógica edge en Workers Static Assets. El default correcto es
repetidamente MENOS edge (A/B client-side, flags build-time, `_redirects` en git, geo por heurística
client) — el valor de la skill son 4 trampas: middleware de Astro no intercepta páginas prerendered;
`run_worker_first` factura cada pageview; Workers Caching cobra assets normalmente gratis; Flagship
sin pricing publicado.

**`auth-simple`.** Skill delgada — la MENOR auth que hace el trabajo, en escalera: nada (default) →
Cloudflare Access a nivel Worker (previews y admin) → service tokens para CI → Supabase Auth (portal
con datos reales). Casi-tombstone: Basic Auth Worker.

**`visual-regression-ci`.** La CUARTA puerta de CI — Playwright `toHaveScreenshot` native-first,
baselines commiteadas y GENERADAS EN CI (el rendering difiere por OS). Tombstones: Lost Pixel
(archivado), BackstopJS (estancado).

**`conversion-patterns`.** CRO estructural bajo medición: secuencia instrumentación-first, árbol de
decisión de conversión primaria (form/WhatsApp/tel), anti-patrones amarrados a reglas de canon. Cero
claims cuantitativos — el dato del propio sitio es la única estadística que acepta.

**`client-discovery`.** La capa cliente: de lo que el cliente dé (conversación, doc, boceto, export
de design-tool) a un brief estructurado con veredicto de factibilidad por ítem. 4 fases: intake
híbrido conversacional-primero → captura por formato → factibilidad por referencia al playbook
(vocabulario cerrado: feasible-as-is / feasible-with-flip / deferred / out-of-scope) → registro de
deferrals en el repo del sitio.

**Recipes de arquetipo** (en el playbook): brand-heavy (PROVEN, el fixture) · lead-gen landing
(DERIVED) · catalog+self-edit (DERIVED) · editorial/story (DERIVED). Frontera documentada, no recipe:
client-portal (el deploy shape cambia de clase).

**`a11y-deep`.** Disciplina WCAG 2.2 AA en TRES capas donde la automatizada es la MENOR: job axe en CI
a ZERO violations · smoke manual codificado por release · audit WCAG-EM reservado para claims
públicos.

## Pins históricos por skill (2026-06-16 — donde choque, manda la tabla vigente de abajo)

| Skill | Pins |
|---|---|
| astro-css-tokens | `astro@6.4.7`, `tailwindcss@4.3.1`, `@tailwindcss/postcss@4.3.0` (NO `@tailwindcss/vite` — #16542), `style-dictionary@5.4.4` (v5), Node ≥22.12 |
| web-security-headers | `astro@6.4.7`, `@astrojs/cloudflare@13.7.0`; SRI opc. `@kindspells/astro-shield@1.7.1`. Review-gate: `staticHeaders` en cada minor del adapter |
| perf-ci-gates | `@lhci/cli@0.15.1` (LH 12.6.1), `treosh/lighthouse-ci-action@v12`, `@biomejs/biome@2.5.0`, `prettier-plugin-astro`, Node 22 |
| seo-aeo-schema | `schema-dts@2.0.0` (TS-only), `@astrojs/sitemap@3.7.3`. Review-gates: profundidad de @graph; deprecación de FAQ |
| motion-system | `gsap@3.15.0` (~45 KB gzip con ScrollTrigger), `@gsap/react@2.1.2`, `motion@12.40.0` |
| webgl-atmosfera | Default = raw WebGL2 vendorizado. Alternativas: `ogl@1.0.11`, `twgl@7.0.0`. WebGPU OUT por fitness |
| signature-anim | `@rive-app/canvas@2.38.1` (default) y variantes hermanas. Re-pin semanal |

## Pins vigentes (2026-08-17 — re-pin audit verificado adversarialmente, aplicados en la fase B0)

| Paquete | Pin previo | Vigente | Nota |
|---|---|---|---|
| `astro` | 6.4.7 | **7.2.2** | 7.0.0 stable desde 2026-06-22; `security.csp` sobrevive el major |
| `@astrojs/cloudflare` | 13.7.0 | **14.2.1** | peer `astro ^7.2.0`; `staticHeaders` sigue sin aterrizar |
| `tailwindcss` | 4.3.1 | **4.3.3** | switch-back a `@tailwindcss/vite` — #16542 cerrado con Astro 7 |
| `style-dictionary` | 5.4.4 | **5.5.1 — JAMÁS 5.5.0** | 5.5.0 = GHSA-xmr7-549p-98w3 (prototype pollution HIGH) |
| `@biomejs/biome` | 2.5.0 | **2.5.8** | patch-only; flag `.astro` sigue experimental |
| `@rive-app/canvas` / `webgl2` | 2.38.1 | **2.40.0** | webgl2 −11.5% de tamaño |
| `@sveltia/cms` | 0.167.2 | **0.191.2** | pre-GA, smoke-test PENDIENTE |
| `motion` | 12.40.0 | **13.1.0** | trae `animateView` (View Transitions) |
| `@kindspells/astro-shield` | 1.7.1 | **RETIRADA** | reemplazada por receta SRI sha384 custom |
| gsap · @gsap/react · schema-dts · @astrojs/sitemap · @lhci/cli · treosh@v12 | — | **sin cambio** | todos siguen latest |

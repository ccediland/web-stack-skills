---
name: bitacora-web-stack-skills
title: web-stack-skills — bitácora
description: >-
  Registro append-only de lo que pasó en este repo y cuándo. Es RASTRO, no fuente: los hechos
  atemporales viven en RESIDENT.md y las reglas en CLAUDE.md; aquí solo se referencian.
last_updated: 2026-08-31
status: vigente
supersede: ninguno
append-only: true
---

# web-stack-skills — bitácora

> **Append-only.** Se agrega al final; **nunca** se edita ni se reordena una entrada pasada. Si algo
> quedó mal, se corrige con una entrada nueva que referencia a la vieja.
>
> Un dictamen o una revisión posterior **es una entrada más, abajo y con su fecha** — jamás una nota
> pegada en esta cabecera. Una cabecera que reinterpreta lo de abajo contradice el append-only, y ya
> pasó una vez: un dictamen afirmaba «la única entrada» sobre un archivo que tenía cinco, y mintió el
> mismo día en que se escribió.

## Para qué sirve tener esto aparte

`RESIDENT.md` contesta *cómo es este proyecto hoy*. Si además carga la narrativa de lo que fue
pasando, cambia en cada sesión — y entonces el knowledge de su Project queda desfasado siempre y el
«Sync now» manual se vuelve diario. Con la bitácora aparte, el RESIDENT solo cambia cuando la forma
del proyecto cambia de verdad, y lo fechado —que es justo lo que envejece el knowledge— nunca entra
ahí.

## Formato

```markdown
## AAAA-MM-DD · <superficie> · <objetivo en una línea>

Hecho: <qué se produjo, en pasado, con cifras si las hay>
Decidido: <IDs de decisiones registradas, o "ninguna">
Pendiente: <lo que quedó abierto>
Siguiente: <la primera acción de la próxima sesión>
```

Superficie es una de: Code, Chat, Chrome, Cowork.

---

## 2026-08-31 · Code · Se siembra la bitácora, con lo que ya había pasado

Hecho: se crea este archivo como parte del doc-set de cinco del estándar de VenturEdge. La primera
entrada no se inventa: recoge los commits reales de esta sesión en el repo.

- gitignore: .mcp.json no se versiona (#3)
- Arreglar front matter y punteros que el auditor encontro (#2)

Decidido: ninguna en este repo (las del sistema viven en `_meta/venturedge-framework/decisiones.md`).
Pendiente: `MAPA.md`, el quinto documento del estándar, que aquí todavía no existe.
Siguiente: al cerrar la próxima sesión que toque este repo, agregar su entrada aquí — sin preguntar.

## 2026-08-31 · Code · Sesión de sistema

Hecho: 3 commit(s) en este repo, dentro de la sesión que cerró el doc-set de cinco en los
quince repos, escribió el criterio de dónde registrar un esfuerzo, y construyó el agregador de
avance sobre los dos dominios de Workspace.

- docs: MAPA.md, el quinto documento del estandar (#5)
- docs: BITACORA.md, el cuarto documento del estandar (#4)
- gitignore: .mcp.json no se versiona (#3)

Decidido: las del sistema viven en `_meta/venturedge-framework/decisiones.md` (`VE-2026-020` a
`VE-2026-033` se escribieron hoy).
Pendiente: lo que quedó abierto está en `pendientes.md` y como issues etiquetados por área.
Siguiente: al cerrar la próxima sesión que toque este repo, agregar su entrada aquí — sin preguntar.

## 2026-06-16 · Code · sesión · Fase 0 — scaffold del repo (migrada del RESIDENT el 2026-09-01)

Hecho: repo público creado; 13 archivos sembrados vía Composio/Git Data API (scaffold `a4bd7f0`,
registro de las 7 primeras skills en `plugin.json` `b24c124`). Bootstrap necesario por un quirk de
GitHub: en un repo recién creado y vacío, la Git Data API (`git/trees`) da 409 "Git Repository is
empty" — se resuelve con un commit inicial vía Contents API y luego se reemplaza el árbol completo.
Decidido: `D-001` (repo público + genérico + MIT, un marketplace con un plugin), `D-002` (8ª skill
diferida como skeleton, fuera del instalable hasta tener sustancia), `D-003` (layout
`skills/<nombre>/` + registro explícito en `plugin.json`).
Pendiente: construir las skills 1–7.
Siguiente: astro-css-tokens (#1).

## 2026-06-17 · Code · sesión · astro-css-tokens (#1) (migrada del RESIDENT el 2026-09-01)

Hecho: decisiones clave — Style Dictionary v5 + `@tailwindcss/postcss` (NO vite, por #16542, con
review-gate) + bridge `@theme inline` sobre capa `:root --ds-*`; OKLCH sin fallback; `$value` string
plano (workaround del bug de composite split #1398/#1494); build por prebuild script; dark = costura
`[data-theme="dark"]`. Bundle SKILL.md + 5 refs. Commit `9b182b0`.

Research verificado contra fuente primaria el mismo día: Style Dictionary v5 (npm 5.4.4) es drop-in
desde v4; Astro 6 × rolldown-vite rompe `@tailwindcss/vite` (#16542, causa raíz #19802
`aliasOnly:true` — el camino que compila es `@tailwindcss/postcss` + `postcss.config.mjs`); OKLCH
`oklch()` es Baseline Widely Available desde 2025-11-09, se shippea sin fallback; el transform SD
`color/css` convierte OKLCH a hex/rgb (para preservarlo, transforms solo-nombre); `@theme inline` no
emite var global (compila a `var(--ref)`, es el mecanismo del bridge) mientras `@theme` plano sí
emite global y `@theme static` fuerza emitir todo; DTCG estabilizó 2025-10-28 y SD da soporte.

## 2026-06-17 · Code · sesión · web-security-headers (#2) (migrada del RESIDENT el 2026-09-01)

Hecho: pivote verificado en source — el adapter CF 13.x NO soporta `staticHeaders`, así que la CSP
estática va por `<meta>` hash-based; `frame-ancestors`/report/sandbox por `public/_headers`;
middleware solo para SSR; hash-only (sin nonce); COEP OFF por default (rompe assets cross-origin de
webgl-atmosfera/signature-anim); SRI = astro-shield ≥1.3.2 solo cross-origin, con review-gate.
Bundle + 5 refs. Commit `4f37a05`.

## 2026-06-17 · Code · decisión · Revisión externa adoptada — D-004 a D-008 (migrada del RESIDENT el 2026-09-01)

Hecho: un dev front-end senior externo revisó el catálogo hasta ese punto. Cinco de sus
recomendaciones se adoptaron.
Decidido:
- `D-004` — el activo durable es el MÉTODO, no las skills: el motor portable es el RESIDENT + la
  cadencia de 5 turnos + el caso contrario + source-priority + el log. Las skills son perecederas y
  reconstruibles desde docs; el `stack-integration-playbook` (entonces diferido) concentra el valor
  irremplazable y su prioridad de llenado sube en cuanto tenga sustancia.
- `D-005` — disciplina de perecederos: cada pin duro y cada workaround por issue-number lleva
  review-gate; "solo + sin auto-update" es un half-life corto, así que se presupuesta mantenimiento
  activo, nunca se asume estabilidad.
- `D-006` — la capa visual (WebGL/Rive) es doble filo: gana el pitch pero arriesga CWV/a11y/lectura
  B2B — UNA por sitio, cuando el brief la justifique, nunca por default.
- `D-007` — Carlos decidió NO a un smoke-test temprano de instalación; §6 del RESIDENT se queda sin
  cambios.
- `D-008` — la tesis comercial/GTM del catálogo vive fuera del repo público, en notas privadas de
  Carlos; el README puede nombrar el nicho como posicionamiento si él lo decide.

## 2026-06-17 · Code · sesión · perf-ci-gates (#3) (migrada del RESIDENT el 2026-09-01)

Hecho: dos puertas de CI — LHCI vía `treosh/lighthouse-ci-action@v12` (reversal: la GitHub App
resultó MÁS integración, no menos), `staticDistDir ./dist/client`, preset `no-pwa`, floors perf 0.9 /
a11y 0.95 en error, mediana de 5 corridas, TBT como proxy de INP — más Biome `ci --reporter=github`
con el flag `.astro` OFF + `prettier-plugin-astro` (reversal: el formatter HTML de Biome es
experimental, no hace match con Prettier). El claim externo "CWV LCP 2.0s" se verificó FALSO — los
umbrales oficiales vigentes son 2.5s/200ms/0.1. Turn 5 recuperado tras un corte de sesión, sin
re-autoría a ciegas. Commit `23d8388f`.

## 2026-06-17 · Code · sesión · seo-aeo-schema (#4) (migrada del RESIDENT el 2026-09-01)

Hecho: el cruce CSP×JSON-LD se descartó como no-issue (`ld+json` es un data block exento de
`script-src`). `schema-dts@2.0.0` vivo pero NO type-checkea `@id` cross-refs — se agregó validación
runtime + script exportado al gate de perf-ci-gates. `llms.txt` se posicionó como B2A/agent-readiness,
NUNCA palanca SEO; `robots.txt` es la superficie real de control de AI-crawlers (16 tokens
verificados). El sitemap solo cubre rutas prerendered; SSR va por endpoint propio. Bundle + 5 refs.
Commit `0b55775`.

## 2026-06-17 · Code · sesión · motion-system (#5) (migrada del RESIDENT el 2026-09-01)

Hecho: árbol de 3 motores native-first — CSS scroll-driven por default (~83% caniuse, no Baseline
pero production-viable) → GSAP+ScrollTrigger solo para scrub/pin/snap/timelines anidadas (~45 KB
gzip real; el claim externo de "7 KB" se verificó falso) → Motion solo en islas React existentes
(variante Mini, 2.3 KB). La CSP resultó favorable: el bundle va hasheado y `element.style` (CSSOM)
queda exento de `style-src`. Se verificó que GSAP es 100% gratis incluyendo ScrollTrigger (el claim
de "Club" de pago se descartó como falso). Bundle + 5 refs. Commit `1bd35f15`.

## 2026-06-18 · Code · sesión · webgl-atmosfera (#6) (migrada del RESIDENT el 2026-09-01)

Hecho: cuatro reversals contra los leans de scoping iniciales — motor final = raw WebGL2 vendorizado
(no OGL como dependencia npm); fallback primario = AVIF/WebP vía `astro:assets` (no un gradiente
CSS); carga = init manual por `requestIdleCallback` + `IntersectionObserver` (el canvas no es
elegible a LCP, así que no usa `client:visible` above-fold); accesibilidad = control de pausa
OBLIGATORIO por WCAG 2.2.2-A. WebGPU quedó OUT por fitness (un solo draw call no usa compute).
Runtime: DPR tope 1.5, ~30fps, pausa offscreen. Bundle + 5 refs. Commit `22f805c5`.

## 2026-06-18 · Code · sesión · signature-anim (#7) (migrada del RESIDENT el 2026-09-01)

Hecho: Rive vía Canvas2D para UN momento interactivo bespoke, con gate de uso DURO (runtime ~840 KB
gzip — solo si hay lógica de estado ramificada real que CSS/GSAP no codifican). CSP load-bearing:
exige `wasm-unsafe-eval` + self-host de WASM y del `.riv` → `connect-src 'self'`. Carga lazy vía
IntersectionObserver + dynamic-import; accesibilidad DIY porque Rive no trae reduced-motion de
fábrica. El proceso de transferencia de archivos cross-environment (Git Data API sin base64) se
resolvió archivo por archivo, con gate de sha-de-blob antes de mover el ref y parenteo sobre el HEAD
vivo. Bundle + 4 refs. Commit `26576412`.

## 2026-06-18 · Code · decisión · cms-self-edit (#8) — selección y build, D-009 (migrada del RESIDENT el 2026-09-01)

Hecho: selección por MÉRITO puro (Carlos descartó el lente de stack-canon: el stack no es fijo).
Ganador Sveltia — CMS git-based client-side puro, montado en `public/admin/index.html` sin backend,
contenido en Markdown/YAML del repo, OAuth GitHub vía el Worker oficial gratis `sveltia-cms-auth`,
media en Cloudflare R2 (SigV4 browser→R2), i18n first-class. Runner-up Pages CMS (magic-link, el
editor no necesita cuenta GitHub, pero reintroduce dependencia de terceros). Etapa 3 (workflow
editorial pesado) = Directus. Descartados con su razón: Keystatic (roto en Astro 6, issues
#1515/#1497 abiertos, sin i18n), EmDash (dev-preview, contenido en DB), Tina (pesado, sin i18n
nativo), Decap (abandonado, Sveltia es su sucesor), Sanity/headless-DB self-host (overkill salvo
Etapa 3). `/admin` se mantiene estático en `public/` para esquivar la CSP `<meta>`; publicación vía
rama `drafts` + check CI. Correcciones de build promovidas a los pins: DeepL deshabilitado en
Sveltia, COOP de `/admin` = `same-origin-allow-popups`. Commits `a8c0948` (selección) / `90110a9`
(bundle) / `d7da1b2`. Con esto, 8/8 skills de la primera ola quedaron autoradas.
Decidido: `D-009` — Sveltia default salvo que los clientes rechacen GitHub como norma (fork abierto,
llamada de Carlos; sin respuesta, queda Sveltia).

## 2026-08-17 · Code · sesión · Análisis pre-sprint y re-scope del plan v1 (migrada del RESIDENT el 2026-09-01)

Hecho: análisis profundo read-only vía Claude Code, 11 agentes, 24 claims load-bearing verificados
adversarialmente (23 confirmados, 1 corregido en atribución). Cubrió: estado del repo, re-pin audit
completo, inventario de `furever-brand`, el contrato `brand-system-skills` 0.6.0, y una auditoría de
solape de triggers entre las 8 skills (3 colisiones ALTAS + un cluster medio de solape en
reduced-motion/pause/lazy-load/scroll-driven/perf-budget; 4 descriptions sin frontera escrita; cero
redirects entre hermanas). Decisiones adjudicadas: migrar a Astro 7 antes de validar (fase B0); la
ingestión de marca se vuelve skill nueva `brand-canon-ingest` (#9, fase B1); el sitio de referencia
es Furever (fase C); el fix-pass de descriptions es prerequisito estructural (fase A). El plan se
re-scopeó por completo (reemplazó `v1-finalization-plan.md`) en la rama `claude/v1-sprint-rescope`.

Hallazgos verificados el mismo día contra fuentes primarias: Astro 7.2.2 es stable (desde
2026-06-22); `@astrojs/cloudflare@14.2.1` exige `astro ^7.2.0`; `security.csp` sobrevive el major
(solo cambios aditivos); Node 20 se dropeó desde Astro 6.1.0, no en 7. Dos citas erróneas cazadas en
`astro-css-tokens` para corregir en B0: Astro 6 nunca usó rolldown-vite (el bug real era npm
hoisteando Vite 8; el issue "#19802" citado es de tailwindlabs/tailwindcss, no de rolldown-vite);
Firefox scroll-driven sigue tras flag en stable. Se confirmó que `furever-brand` ya habla el contrato
de tokens (proyección string pre-emitida para SD v5), con delta real en los 4 schemes (objetos OKLCH
compuestos sin proyección string, a serializar con un patrón C-1 nuevo).

## 2026-08-17 · Code · sesión · Fase B0 — migración a Astro 7 (migrada del RESIDENT el 2026-09-01)

Hecho: Fase A (cleanup/docs) mergeada a main primero (PR #1, `d1591e0`). Re-pin lockstep aplicado
skill por skill, un commit por skill: astro-css-tokens a astro 7.2.2 + tailwind 4.3.3 + switch-back a
`@tailwindcss/vite` (#16542 ya cerrado; las 2 citas erróneas corregidas) + Style Dictionary 5.5.1
(jamás 5.5.0); web-security-headers al adapter 14.2.1 (el gate de `staticHeaders` seguía sin
disparar) + retiro de `astro-shield` reemplazado por receta SRI sha384 custom + watch nuevo sobre el
issue #16692; perf-ci-gates a Biome 2.5.8 (flag `.astro` sigue OFF); motion-system a Motion 13.1.0
(Firefox ~85.4%, watch a Fx156 ~oct-2026); signature-anim a Rive 2.40.0; cms-self-edit a Sveltia
0.191.2 (el review-gate de Keystatic seguía sin cumplirse). Baseline 7.2.2/14.2.1 replicado a
seo-aeo-schema y webgl-atmosfera.

El smoke scaffold sobre Astro 7 real cazó un bug de receta y lo corrigió: `@tailwind base` es no-op
SILENCIOSO en Tailwind v4 — el build pasa pero Tailwind nunca procesa el CSS (cero utilities
generadas, la línea literal llega al browser). El entry stylesheet debe abrir con `@import
'tailwindcss'`. Confirmado en el mismo build: CSP meta hash-based en página estática, utilities
compilando a `var(--ds-*)`, OKLCH preservado, dark override funcionando, y el adapter SÍ inyectó
`Cache-Control: immutable` para `/_astro/*` en la config default (watch #16692 OK en este punto).

## 2026-08-17 · Code · sesión · Fase B1 — brand-canon-ingest (#9) (migrada del RESIDENT el 2026-09-01)

Hecho: scope lockeado en chat, verificado contra `furever-brand` real ANTES de autorar — `run-gates.mjs`
salió ALL-GREEN (19 PASS + 1 NOT-RUN honesto, red-team live diferido por diseño), 4 schemes × 53
roles, contrato 0.6.0 pineado al commit `abcc31f` del tool-repo. Cero reversiones de arquitectura; 3
ajustes menores documentados: orden de emisión del serializador (`:root` primero, empate de
especificidad — si no, dark gana en modo light), la fila de projections queda machine-checked bajo
R6a, y la cita exacta del veto `-fc` es `G-LOGO-02`. Bundle autorado bajo skill-author +
md-house-style, `quick_validate` PASS, evidencia dura de los 2 artefactos de código (el serializador
corrido contra 212 vars del repo real; el transform de cubicBezier compilado en SD 5.5.1). Frontera
de marca añadida a la description de astro-css-tokens (711 chars). Chequeo de misroute contra las 8
hermanas + 2 plugins de brand-system: sin canibalización bidireccional. La ratificación del Stage-10
del canon NO quedó registrada en esta sesión (el prompt no abrió con la frase-mecanismo) — se deja
pendiente para Carlos. Commits `4e76123` (bundle+registro) y `b19b693` (frontera en #1).
Pendiente: ratificación Stage-10 del canon — a cargo de Carlos.

## 2026-08-17 · Code · sesión · Fase B2 — validación mecánica (migrada del RESIDENT el 2026-09-01)

Hecho: bajo el plan re-scopeado (`execution-plan.md`, reemplazó y eliminó `v1-finalization-plan.md`).
Descongelamiento de descriptions: 7 pasaron de "Astro 6" a "Astro 7" (el censo real fue 7, no 6 —
webgl-atmosfera seguía en 6 por un revert deliberado de B0), más el recorte de seo-aeo-schema
(1023→1015 chars) y un fix de "v13" obsoleto en web-security-headers; tras el triggering se agregó
un hook de "dark mode" en astro-css-tokens (750 chars). Las 9 skills pasaron `quick_validate` +
`package_skill` (scripts canónicos descargados de `anthropics/skills@main`). Install real probado en
scopes `user` y `project` vía marketplace de path local (lee el checkout en vivo — `claude plugin
marketplace add <path> --scope`), 9 skills descubiertas. Triggering: 30 prompts × 10 jueces ciegos,
30/30 sin misfires — todos los prompts obligados rutearon bien; un probe cross-plugin ruteó correcto
(narrow) hacia una hermana externa (`brand-canon-builder`) y se registró como upstream suggestion:
su description ganaría un ancla explícita tipo "for a NEW brand (no existing token source)" — el fix
es de su lado, no del nuestro. Smoke de Sveltia 0.191.2 PASS (screenshot verificado). Manifiestos a
`1.0.0-rc`.

Lección reproducible: el triggering con jueces ciegos escala — 30 prompts × 10 agentes independientes
que solo ven las descriptions, sin misfires, y los márgenes narrow señalan fronteras reales (no
fallas) cuando el runner-up es el correcto. La batería y la metodología se reusaron en cada ola
posterior (W1–W4).

## 2026-08-17 · Code · sesión · Fase C — fixture Furever, primer build compuesto real (migrada del RESIDENT el 2026-09-01)

Hecho: ejecutada en la rama `claude/c-fixture` de `furever-web` (el repo YA EXISTÍA con producción
viva en su main — se trabajó en rama, con la regla dura de NUNCA mergearla a main; deviations
documentadas). Ingest de marca vendorizado al commit `b0a5332` (gates ALL-GREEN como paso 0, 23
artefactos, manifest de procedencia). 8 skills compuestas; la capa visual fue webgl-atmosfera
(evidencia de canon: `ALGO-ATMOSPHERE-COMPOSE`, y cero `.riv` en el repo de marca). Ambas puertas de
CI en verde sobre el build real; preview vivo verificado (headers, CSP meta, immutable, dark switch,
admin); fila de projections actualizada con R6a PASS (`furever-brand@5d99526`). 3 fixes de recetas
commiteados en `claude/c-fixture-fixes`. Nota: la decisión de junio del sitio real de NO usar Astro
(su propio Decision registry) queda registrada y NO superseded — el fixture es insumo de prueba por
orden del owner, no un cambio de esa decisión.

Veinte lecciones cazadas, las que importan para cualquier build futuro:
1. Los `style=""` estáticos quedan BLOQUEADOS bajo CSP hash-based (84 violaciones en vivo → 0 tras
   el fix): los hashes cubren `<style>` y scripts bundled, NUNCA atributos — el CSSOM de JS (GSAP
   `el.style.x=`) sí está exento. Todo el styling va por clases.
2. `_headers` CONCATENA cabeceras homónimas entre reglas (COOP duplicado en `/admin/` visto en
   vivo) — el override real es `!` (detach) antes de re-declarar.
3. El script inline pre-paint (no-flash) necesita hash manual single-sourced: una constante
   importada por config Y por layout — editarla sin re-hash rompe silencioso.
4. Sitio 100% prerendered + adapter 14.2.1 → `dist/server` VACÍO → Worker assets-only (sin
   `main`); el adapter emite su propio `dist/client/wrangler.json`.
5. Workers Builds branch previews son deploy temporal nativo sin auth local de wrangler: el
   `build.command` corre dentro de `versions upload`; prod solo se mueve con `versions deploy`.
6. El watch del issue #16692 no dispara: `Cache-Control: immutable` de `/_astro/*` se verificó en
   vivo bajo config default.
7. Biome sobre un repo con marca ingerida exige `css.parser.tailwindDirectives` + excludes (SVGs
   exact-file, CSS generado por SD, `.wrangler/`).
8. La vendorización en ingest-time funciona limpia: CI standalone sin repo de marca,
   `run-gates.mjs` como precondición dura, manifest de procedencia barato y completo.
9. Shader × marca: el hex del spine de schemes es la fuente numérica para uniforms GL (CSS sigue
   OKLCH-only); swap de paleta por MutationObserver de `data-theme`.
10. Shiki (code fences en markdown) emite estilos inline anti-CSP — usar Prism o sin código; watch
    para blogs técnicos.
11. `astro preview` (workerd) muere si se rebuildea `dist` debajo — hay que reiniciarlo antes de
    re-probar.
12. `site` (canonical/OG/sitemap absolutos) apunta al workers.dev estable, no al alias del preview
    — en sitios reales `site` va por environment.
13. Los 5 woff2 de la marca (~145KB) rozan el budget de fuentes de 150KB — presupuestar por brief,
    no por default.
14. El visual se eligió por EVIDENCIA de canon, no por gusto — fabricar un `.riv` inexistente
    habría violado owner-decides.
15. LA GORDA: LHCI 0.15 rechaza `assertions` + `budgetsFile` juntos y `treosh@v12` SE TRAGA el
    crash del assert — el job sale VERDE habiendo asertado NADA (gate vacuo, cazado por verificación
    adversarial, no por el CI). Fix: budgets vía assertions `resource-summary:*`; hábito nuevo —
    confirmar en el log que "Checking assertions against N URL(s)" corrió.
16. Páginas app-shell (el admin del CMS) revientan los floors de Lighthouse — se sacan del set
    auditado vía `collect.url`.
17. Los twins de scheme con `display:none` SE DESCARGAN aunque lleven `loading=lazy` (Chromium) —
    para pares logo light/dark, inline verbatim (`?raw`), cero requests.
18. Contraste × canon: el par action/action-text de lightA da 3.94:1 (pasa como large text, umbral
    3:1) con la paleta intacta; `text-subtle` (3.45:1) es rol de hint, nunca de copy real.
19. `git add -A` tras corridas de herramientas es trampa: la primera corrida de LHCI (Chrome de
    Windows desde WSL) dejó rutas UNC que se colaron a dos commits — gitignorear los outputs de
    herramientas antes de correrlas.
20. WebGL decorativo × software GL: en runners sin GPU el shader se renderiza por CPU (SwiftShader)
    y el rAF se vuelve TBT de 145s — invisible en local con GPU. `failIfMajorPerformanceCaveat: true`
    NO basta (verificado — TBT idéntico con el flag). Fix de dos capas: probe de
    `WEBGL_debug_renderer_info` (rechaza swiftshader/llvmpipe) + watchdog de frame-cost (10 frames
    >25ms promedio → mata el canvas, queda el fallback de tokens).

## 2026-08-17 · Code · sesión · Fase D — playbook y ship v1.0.0 + W1 Run A (migrada del RESIDENT el 2026-09-01)

Hecho: `stack-integration-playbook` promovida de `deferred/` a `skills/` (git mv) y autorada en
cadencia comprimida, sin re-derivar investigación — research = lo ya aprendido en las sesiones
anteriores + la recipe del fixture. Bundle: SKILL.md (el orden canónico
ingest→tokens→security→gates→seo→motion→UN visual→cms como cadena de dependencias, no ritual —
omitir pasos es Lego, reordenarlos no) + `seams-and-gotchas.md` (9 seams primarios y sus modos de
fallo probados) + `stack-integration-map.md` (mapa de puntos de contacto, no tutoriales) +
`recipes.md` (recipe #1 + skeleton-contrato para arquetipos futuros). Registrada 10ª en
`plugin.json`, `quick_validate` 10/10. Triggering 16/16 con 4 jueces ciegos sobre superficie de 12
descriptions — 4 prompts de composición rutearon CLEAR al playbook, 11 de capacidad a su dueña, 1
ambiguo rutéo a NONE (sano). Ship: manifiestos a 1.0.0, README v1 corregido (brand-canon-ingest
faltaba en la tabla), tag anotado `v1.0.0` + GitHub Release. Nota de mantenimiento cazada aquí por el
verificador pre-merge, resuelta después en W5: el plugin usaba `source: "./"`, así que el paquete
instalado arrastraba RESIDENT.md/execution-plan.md/CLAUDE.md del repo a cada instalación (~90 KB de
docs internos, no dañino pero sucio).

W1 Run A (solo análisis, sin build): 2 agentes de research sobre fuentes primarias (docs oficiales,
npm, Context7) + 1 verificador adversarial sobre los claims load-bearing. Decision sheet entregada:
`data-layer` con 4 forks (default = custom loader build-time; rebuild = Supabase webhook → Workers
Builds Deploy Hook directo; typing = zod único como contrato; llaves nuevas de Supabase);
`forms-lead-system` veredicto RECIPE del playbook, no skill propia (una sola ruta nativa dominante:
Action + Turnstile + Supabase insert-only + Resend, con flip conditions nombradas). Hallazgos
durables: Workers Builds tiene Deploy Hooks nativos desde 2026-04; las live content collections son
estables pero exigen on-demand; Astro Actions estables con RPC desde páginas prerendered;
`@astrojs/db` deprecada y removida en Astro 7 (tombstone); Supabase migró a llaves
`sb_publishable_`/`sb_secret_` (las legacy mueren fin de 2026).

## 2026-08-17 · Code · sesión · W1 Run B — data & captura, ship v1.1.0 + W2 Run A (migrada del RESIDENT el 2026-09-01)

Hecho: check pendiente resuelto primero (timebox 15 min) — Cloudflare Email Service vs Resend para
la notificación al dueño: Email Service GANA por native-first (beta pública desde 2026-04-16;
verified-destination gratis en TODOS los planes; MailChannels muerta desde 2024); Resend queda
documentado como flip. `data-layer` autorada (11ª skill, SKILL.md + 4 refs) desde el research
lockeado, sin re-derivar. `forms-lead-recipe.md` nueva en el playbook con los 2 gotchas de campo
cazados en el fixture (split-config, `cloudflare:workers` env). Fixture extendido en
`claude/c-fixture`: proyecto Supabase sintético `furever-fixture` ($0 confirmado), tablas
planes/leads con RLS de la recipe, catálogo graduado seed→Supabase build-time, `/contacto` completo,
split-config resuelto tras cazar el fail real de Workers Builds, Deploy Hook por API + webhook
`pg_net` + Vault. Verificación viva: POST 200 en el deploy, browser E2E sin errores de consola, ciclo
webhook end-to-end, ambas puertas de CI verdes. Triggering 24/24. Ship v1.1.0.

Lecciones de campo verificadas EN VIVO: mixed prerendered+on-demand es first-class en Workers Builds
(el fixture pasó de assets-only a assets+`main`, con solo `/_actions/*` on-demand); el split-config
real es que wrangler exige `main` en el config que parsea pero el Vite plugin de Cloudflare truena
con un `main` que aún no existe durante `astro build` — se resolvió con `configPath` apuntando a un
`wrangler.vite.jsonc` mínimo sin `main`, dejando el `wrangler.jsonc` raíz como config de deploy;
`Astro.locals.runtime.env` está REMOVIDO desde Astro v6 (secretos vía `import { env } from
'cloudflare:workers'`, falla solo en runtime workerd — hay que probar contra `astro preview`, no
solo compilar); Turnstile convive con la CSP hash-based por allowance de origen puro, verificado con
dummy keys contra el deploy vivo; el ciclo webhook→Deploy Hook completo tarda ~2.5 min; sembrar el
seed sin acentos y las rows de Supabase con acentos hace legible a simple vista qué fuente construyó
cada deploy; el `checkOrigin` de Astro rechaza POSTs cross-site a `/_actions` (protección gratis,
pero el smoke por curl necesita el header `Origin`); `networkidle` nunca llega con Turnstile en la
página (sockets vivos) — hay que esperar por `domcontentloaded` + condición explícita.

W2 Run A (análisis, sin build): 3 briefs de research (i18n / media / a11y) + verificación adversarial
11/12 confirmados (1 corregido: el default de `imageService` cambió en el adapter v13.0.0, no v14).
Check local del contrato de marca: 0.6.0 no define tokens multi-locale.

## 2026-08-17 · Code · sesión · W2 Run B — alcance y pulido, ship v1.2.0 + W3 Run A (migrada del RESIDENT el 2026-09-01)

Hecho: 3 skills nuevas desde los briefs del Run A recuperados verbatim, sin re-derivar —
`i18n-system` (Astro core i18n, default locale en raíz, diccionario tipado zero-dep, colecciones en
`multiple_folders` compartiendo `translationKey`/`canonical_slug` con Sveltia, hreflang con
`x-default`, sin detección/redirect por default) + `media-optimization` (`imageService: 'compile'`
explícito — el default del adapter cambió a `cloudflare-binding` en v13.0.0 y ese free tier revienta
cerrado a 5k uniques/mes; `priority` solo en la imagen LCP; video en tiers forzados por plataforma) +
`a11y-deep` (3 capas: job axe en CI a cero violaciones, smoke manual codificado, audit WCAG-EM
reservado para claims públicos). Descriptions 1001/1011/1017 con fronteras escritas. Fronteras
cruzadas re-autoradas: seo-aeo-schema (emisión de hreflang) y perf-ci-gates (aloja el job axe).
Registradas en `plugin.json`, validate 14/14. Fixture: `/en/` sintética con hreflang/x-default
verificado en el deploy, blog en `multiple_folders`, tercera puerta axe (8 páginas, 0 violations),
LHCI a 6 URLs, burn E2E en browser real sin errores. Gotcha nuevo: AxeBuilder rechaza páginas de
contexto implícito, exige `browser.newContext()`. Triggering 28/28 con 7 jueces sobre superficie de
16 — las fronteras nuevas rutearon en ambas direcciones (perf se auto-excluye de "budget de imagen",
a11y-deep se auto-excluye de "subir el floor de Lighthouse"). Ship v1.2.0. W3 Run A: 2 briefs
(edge-logic / analytics) + verificación adversarial, decision sheet al chat.

## 2026-08-17 · Code · sesión · W3 Run B — edge y medición, ship v1.3.0 + W4 Run A (migrada del RESIDENT el 2026-09-01)

Hecho: `edge-logic` autorada (#15, delgada — un solo SKILL.md, cero references, el peso no las
pidió): el default correcto es repetidamente MENOS edge (A/B client-side, flags build-time,
`_redirects` en git, geo por heurística client), con 4 trampas documentadas — middleware de Astro NO
intercepta páginas prerendered, `run_worker_first` factura cada pageview contra el cap de 100k/día,
Workers Caching cobra assets normalmente gratis, y Flagship (beta) no tiene pricing publicado.
`analytics-recipe.md` nueva en el playbook: Umami como default de eventos (2.3 KB, MIT, self-host en
PikaPods), Cloudflare Web Analytics re-encuadrado como capa RUM ambiental (no puede ser default de
medición — cero custom events), vocabulario estándar de eventos (`lead_submit` server-side desde el
Action), CSP exacta, env-gate doble, tombstone de GA4 salvo Google Ads. Fixture: A/B client-side
sintético verificado E2E en browser real (asignación pre-paint, persistencia, flip forzado, 0
errores); analytics como stub honesto env-gated, con el smoke aseverando la AUSENCIA del tag además
de las presencias. Triggering 28 prompts × 8 jueces, 27/28 a la primera + 1 gap léxico real
(RUM→NONE, corregido con 3 recortes + 1 hook, re-test 3/3). Ship v1.3.0.

Lección de campo: un run de CI cancelado por concurrency (push de docs mientras LHCI corría) se lee
igual que uno fallido si no se mira la causa — el veredicto de las puertas siempre es sobre el run
del HEAD final. W4 Run A (triage tier 3, sin build): 3 agentes de research (view-transitions+
speculation-rules / auth+visual-regression / dotLottie+LFPDPPP) + verificador adversarial sobre 14
claims. Tabla de triage al chat: 3 skills propias delgadas, 4 absorciones a skills existentes, 2
recipes nuevas del playbook, 1 frontera actualizada (Rive movió sus exports a Cadet $9/seat, el
runtime sigue MIT).

## 2026-08-18 · Code · sesión · W4 Run B — tier 3, ship v1.4.0 + W5 Run A (migrada del RESIDENT el 2026-09-01)

Hecho: lock del triage aplicado tal cual — 3 skills delgadas (`auth-simple`: escalera
Access-Worker-level/tokens/Supabase Auth, ancla con días de edad al autorar; `visual-regression-ci`:
cuarta puerta de CI vía Playwright screenshot, baselines nacidas en CI; `conversion-patterns`: CRO
estructural, sobrevivió su review-gate de sustancia con cero claims cuantitativos), 4 absorciones
como extensiones versionadas (view transitions + hover micro-interactions a motion-system; content
modeling a cms-self-edit; component conventions a astro-css-tokens; frontera Rive-vs-Lottie
actualizada en signature-anim), 2 recipes nuevas en el playbook (speculation-rules, legal/LFPDPPP).
Fixture: cuarta puerta visual con `reducedMotion: 'reduce'` como mecanismo de determinismo (los
fallbacks de accesibilidad SON el estado fotografiable estable), theme morph con view transitions
nativas, pase estructural de conversión que encontró 3 huecos reales (nav sin link a la página de
conversión, sin ask de cierre, teléfono sin `tel:`), página de aviso de privacidad desde el template
legal, auth como STUB honesto (Access exige acción del owner). Triggering 31/32 a la primera + 1 gap
léxico (Rive-vs-Lottie→NONE, corregido, re-test 3/3). Ship v1.4.0.

Lección de proceso: el review-gate de sustancia declarado ANTES de autorar disciplina la autoría —
conversion-patterns se autoró bajo amenaza real de degradación y salió más honesto por ello. W5 Run
A: research puntual (installer bloat verificado contra docs primarias de plugins: NO existe ignore
mechanism, el patrón es mover el source a subdirectorio) + decision sheet de la ola final al chat.

## 2026-08-18 · Code · sesión · W5 Run B — capa cliente, ship v2.0.0, cierre del roadmap (migrada del RESIDENT el 2026-09-01)

Hecho: la ola final, locks tal cual, cero reversiones. `client-discovery` autorada (#19, 4 fases:
intake híbrido conversacional-primero con banco de preguntas por 6 dominios → captura por formato
(escrito/boceto/export de tokens) → factibilidad por referencia al playbook con vocabulario cerrado
de veredictos → registro de deferrals en el repo del sitio, no en el catálogo). Burn retroactivo
real: el material de Furever (brief de experiencia + repo de marca) corrido por las 4 fases produjo
un worked example sanitizado dentro de la skill. Playbook: 3 recipes de arquetipo bajo un contrato
evolucionado con dos estatus honestos (PROVEN el brand-heavy del fixture; DERIVED lead-gen landing,
catalog+self-edit, editorial/story), frontera client-portal documentada sin recipe, poda de
"consultoría-AI" (cero skills exclusivas). Maturity del marketplace: política de versionado escrita
en CLAUDE.md, calendario de drift de 17 watches, restructure a `plugin/` como único subdirectorio que
viaja al installer (no existe ignore mechanism en plugins — el fix descubrió además que la LICENSE
deja de viajar si se filtra sin cuidado, y MIT exige que la acompañe), `upstream-suggestions.md` con
S-1 a S-4. El fixture se declaró banco permanente (documentado en su propio RESIDENT). Validate+
package 19/19, install re-test 19/19 con cache limpio (944 KB solo-skills+LICENSE). Triggering final
32/32 a la primera (8 jueces ciegos, superficie 21, cero misfires; los 5 NARROW fueron todos
fronteras por diseño con runner-up correcto). Ship v2.0.0 ("Full catalog"); el plan se archivó a
`archive/execution-plan-2026-08.md` con estado ARCHIVED — CLOSED AT v2.0.0. Con esto el RESIDENT de
entonces quedó como única fuente viva del catálogo, y el catálogo entró a modo mantenimiento (el
trabajo futuro corre por el calendario de drift y por burns de skills nuevas contra el fixture).

## 2026-08-18 · Code · sesión · Burns reales — Cloudflare Access y Email Service (migrada del RESIDENT el 2026-09-01)

Hecho: los dos primeros "burns" contra infraestructura real del owner, no simulada.

Access (auth-simple): el modelo real de Worker-level Access es de DESTINATIONS con precedencia
estrecha-sobre-ancho — `worker` (worker_id) protege TODO lo ruteado al Worker, custom domains de
producción incluidos; `preview_worker` solo los preview URLs; `public` (host+path con wildcards)
solo esa ruta, y estos dos últimos toman precedencia sobre `worker`. La trampa cazada: el Worker del
fixture sirve producción (furever.com.mx) además de los previews — "atar Access al Worker", que la
skill enseñaba como mecanismo único, habría gateado producción. Se corrigió a app `public`
path-scoped sobre el host del preview. También se corrigió que OTP NO viene preinstalado en orgs
nuevas (alta explícita de IdP) y que un Access app sobre host workers.dev se crea 100% por API
account-level sin referencia a zona. Verificado en vivo: 302 al login en `/admin*`, OTP end-to-end,
cookie `CF_Authorization` de 24h, rutas hermanas y las 4 puertas de CI intactas.

Email Service (forms recipe): se corrigió que exige DOS prerequisitos owner-side, no uno — destino
verificado Y from-domain onboarded a Email Sending (subdominio propio de envío con SPF/DKIM, sin
tocar el MX/SPF del apex — convive con Google Workspace en la misma zona, probado vivo). Binding
mínimo `{"name":"EMAIL"}`; el send es un objeto plano con `from.email` (el REST usa `address` — no
mezclar). El primer envío real aterrizó en SPAM del inbox verificado — esperable con reputación cero
de un subdominio recién nacido; hallazgo fechado, registrado en la recipe.

## 2026-08-18 · Code · sesión · Acciones del owner cerradas — ratificación de marca, Umami, cierre de los burns (migrada del RESIDENT el 2026-09-01)

Hecho: cuatro pendientes que solo Carlos podía cerrar, cerrados el mismo día.
- Ratificación Stage-10 del canon v2: Carlos ratificó por directiva en chat; registrada en
  `furever-brand` por el mecanismo propio del contrato (`sources/ratification—2026-08-18.md`,
  hasheada en CHECKSUMS.txt), stamps "pending" volteados en `canon.json` + 4 capas, board re-corrido
  ALL-GREEN. El fixture ya consume un canon ratificado.
- Cuenta Umami + site id: Carlos creó la cuenta (Cloud free) y el id se cableó por la vía diseñada
  (`PUBLIC_UMAMI_ID` en build.command, `UMAMI_WEBSITE_ID` como Workers var). Primera entrega en vivo
  verificada: pageviews + `plan_view` + `tel_click` + `ab_expose` + `whatsapp_click` aceptados por el
  collector, `lead_submit` server-side confirmado en el dashboard. Dos correcciones de campo a la
  recipe: el tracker de Umami Cloud postea a `gateway.umami.is/api/send` (la CSP original solo
  permitía `cloud.umami.is` y bloqueaba la entrega); el bot-check de Umami descarta UAs no-browser en
  silencio, así que el send server-side debe reenviar el User-Agent del cliente.
- Zero Trust/Access (burn f): cerrado — ver la entrada de burns de este mismo día.
- Onboarding de Email Service (burn g): cerrado — ver la entrada de burns de este mismo día.

Pendiente: gobernanza del owner que sigue abierta — GAP-006 (banco de imagen), GAP-014 (aviso de
privacidad real), doc de licencia BTN; decidir si las upstream suggestions S-1..S-4 se convierten en
issues de `brand-system-skills`.

## 2026-09-01 · Code · sesión · Mudanza de contenido a la base v2 del estándar de documentos (migrada del RESIDENT el 2026-09-01)

Hecho: RESIDENT.md (416 líneas, 76 fechas) se compactó a la base v2 del estándar de documentos —
front matter al esquema base (name/title/description/last_updated/status/supersede), sección
`## Rumbo` nueva de tres partes, y toda la narrativa fechada (§8 Descubrimientos del proceso, §9
Estado, §11 Log de sesiones, más las decisiones de §7 con ID nuevo `D-001` a `D-009`) mudada aquí
como 22 entradas cronológicas. El detalle técnico atemporal que era demasiado largo para el
presupuesto (veredictos extendidos por skill, tablas de pins, el calendario de drift de 24 watches)
se movió a `docs/skill-verdicts.md` y `docs/calendario-drift.md`, referenciados desde el RESIDENT.
CLAUDE.md se reordenó para abrir con «cómo verificar» (`python3 ~/.claude/tools/audita.py revisa .`
es el único verde real del repo — no hay build ni tests).
Decidido: ninguna nueva (la mudanza no cambia arquitectura, solo la casa del contenido).
Pendiente: ninguno nuevo — `python3 ~/.claude/tools/audita.py revisa .` corre limpio salvo los 3
avisos de `archive/execution-plan-2026-08.md` (evidencia, no se corrigen).
Siguiente: al cerrar la próxima sesión que toque este repo, agregar su entrada aquí — sin preguntar.

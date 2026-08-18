# Upstream suggestions — brand-system-skills

> Registro formal de mejoras que este catálogo detectó y cuyo fix vive en `brand-system-skills` (otro repo, otro plan). Este repo JAMÁS edita aquel: cada entrada queda aquí con su evidencia y el owner decide si la convierte en issue/cambio allá. Una entrada se cierra cuando el cambio upstream shippea (o el owner la rechaza) — se marca, no se borra.

## S-1 — Stamp de versión de contrato en los repos emitidos

- Detectada — 2026-08 (Phase B1/C, `brand-canon-ingest`).
- Evidencia — los repos de marca emitidos NO llevan stamp de qué versión del contrato hablan; el contrato se auto-impone por los gates copiados, pero el consumidor (skill #9) tuvo que pinear contrato 0.6.0 + sha del tool-repo por fuera. El hop C-1→web ya se rompió ~6 semanas una vez sin que nada lo declarara.
- Sugerencia — el builder escribe un `contract-version` (o campo en un manifest existente) en cada repo emitido; los gates copiados lo verifican contra sí mismos.
- Beneficio — el consumidor detecta drift de contrato leyendo el repo de marca, sin pin externo ciego.

## S-2 — Proyección string de schemes emitida por el builder

- Detectada — 2026-08 (Phase C, ingest real de `furever-brand`).
- Evidencia — `tokens/web/{base,semantic,component}.json` llegan como proyección string lista para Style Dictionary, pero los 4 schemes (53 roles c/u) llegan como objetos OKLCH compuestos SIN proyección — el consumidor vendoriza ~50 líneas de serializador (patrón C-1 de `tools/tokens-project.mjs`) para cerrar el hueco, con byte-parity que hay que mantener a mano.
- Sugerencia — el builder emite también la proyección CSS de schemes (mismo serializador que ya vive en su tool-repo), y la skill de ingest la consume verbatim como hace con los tokens.
- Beneficio — desaparece el artefacto vendorizado más frágil del cruce (drift de serializador = bug silencioso de theming).

## S-3 — Ancla "for a NEW brand" en la description de brand-canon-builder

- Detectada — 2026-08-17 (triggering de B2, juez ciego).
- Evidencia — "set up design tokens for our new brand" ruteó CORRECTO a `brand-canon-builder` pero con margen narrow frente a `astro-css-tokens`; el ancla que decidió fue "for our new brand" + el disclaimer de palette del lado nuestro. Sin fix posible de nuestro lado sin inflar la description de #1.
- Sugerencia — la description del builder gana un ancla explícita tipo "set up design tokens for a NEW brand (no existing token source)".
- Beneficio — el par builder↔tokens deja de depender de un disclaimer del lado consumidor.

## S-4 — Deslinde site-brief en la description de brand-canon-scoper

- Detectada — 2026-08-18 (W5, autoría de `client-discovery`).
- Evidencia — la description del scoper abre con "interview the owner" y "scope … in conversation" sin deslinde hacia briefs de SITIO; con `client-discovery` en el catálogo, el par "brand discovery" ↔ "client/site discovery" queda deslindado solo del lado nuestro ("a SITE brief, not a brand brief — … goes to brand-canon-scoper"). La frontera bilateral es el patrón que este catálogo probó estable (W2: hreflang; W4: absorciones); unilateral funciona pero con margen más frágil.
- Estado de evidencia — el triggering final de v2 juzga el par en ambas direcciones; esta entrada carga el veredicto cuando exista.
- Sugerencia — el scoper gana una frase espejo tipo "scopes the BRAND, not a website project — client intake and site briefs belong to the site pipeline".
- Beneficio — el par rutea estable desde ambos lados, como todas las fronteras bilaterales del catálogo.

---
title: web-stack-skills — calendario de drift
summary: Índice de los 18 disparadores de obsolescencia que vigilan el catálogo en modo mantenimiento — versiones, pricing y reformas legales que, al moverse, exigen re-verificar una skill o una recipe contra su fuente primaria.
last_updated: 2026-09-01
applies_to: web-stack-skills
---

# web-stack-skills — calendario de drift

Cadencia: revisión TRIMESTRAL de toda la tabla + atención on-changelog-hit fuera de ciclo cuando un
watch dispara. Un run de drift = rama `claude/drift-<YYYY-Qn>`, re-verificación de los watches
disparados contra fuente primaria, burn en el fixture cuando el cambio toque una receta ejercitada, y
release PATCH (o MINOR si abre superficie) bajo la política de versionado del `CLAUDE.md`. Los pins
finos por paquete viven en `skill-verdicts.md` y en cada `SKILL.md`; esta tabla es el ÍNDICE de lo
que envejece.

| Watch | Dueña | Trigger esperado | Acción al disparar |
|---|---|---|---|
| Worker-level Access — madurez/pricing/docs (EL MÁS VOLÁTIL del catálogo) | auth-simple | changelog de Cloudflare Zero Trust | PRIMER BURN HECHO (2026-08-18: destinations + OTP + path-scoped). Re-verificar destinations/pricing/semántica en cada hit |
| Email Service beta → GA (quotas, SLA, deliverability del subdominio) | forms recipe (playbook) | changelog de Email Service | re-verificar límites y prerequisitos; primer envío real aterrizó en spam — re-checar reputación al GA |
| Flagship — pricing sin publicar (empatado en volatilidad) | edge-logic | anuncio de pricing/GA | resolver el review-gate del fork de flags; puede mover el default |
| Reforma LFPDPPP en Congreso (~sept-2026) | legal recipe (playbook) | publicación en DOF | re-verificar template completo + short notice + no-banner; PATCH inmediato |
| Firefox scroll-driven sin flag (Fx156 ~oct-2026) | motion-system | release notes de Firefox | actualizar el caveat NO-Baseline del motor CSS; puede subir el default |
| Majors de Astro (~12–18 meses; el adapter movió API en ambos majors vigentes) | TODO el catálogo | astro@8 alpha | presupuestar ola de re-pin tipo B0 — fase propia, no un patch casual |
| Sveltia GA (milestone 1.0 RC) | cms-self-edit | release GA | quitar el caveat pre-GA; smoke del admin; revisar Editorial Workflow |
| LHCI × Lighthouse 13 (bloqueado por Node 22.19+) | perf-ci-gates | release de @lhci/cli con LH13 | re-pin + revisar audit llms.txt nuevo en el gate |
| Speculation Rules → Baseline | speculation recipe (playbook) | dashboard Baseline | la escalación deja de ser Chromium-only |
| `experimental.clientPrerender` estable | speculation recipe (playbook) | release notes de Astro | reconsiderar enseñarlo como vía integrada |
| Trenes Rive (~semanal) + madurez dotLottie state machines | signature-anim | al tocar la skill | re-pin del runtime; re-juzgar la frontera Rive-vs-dotLottie |
| CF Web Analytics custom events ("Not yet") | analytics recipe (playbook) | changelog de CF | re-encuadre posible del RUM ambiental hacia eventos |
| Majors de Umami + pesos de scripts (tabla de evidencia FECHADA) | analytics recipe (playbook) | major de Umami | re-medir pesos — la tabla es evidencia, no dogma |
| Supabase legacy keys mueren (fin 2026) | data-layer | aviso de Supabase | ya en llaves nuevas; verificar que ejemplos/docs no regresen a legacy |
| Keystatic review-gate (#1497 abierto) | cms-self-edit | cierre de ambos issues | reconsiderar en la escalera (el cero-i18n sigue siendo tapón MX) |
| WebGPU Baseline | webgl-atmosfera | Baseline MDN + necesidad de compute | reabrir el veredicto de motor |
| Contrato brand-system (0.6.0 @ tool-repo) | brand-canon-ingest | release del contrato | re-pin de las 4 superficies consumidas + re-correr projections R6a |
| Re-medición del fixture (Supabase free pausa a ~1 semana idle) | fixture furever-web | cada run de drift | CI es inmune (builds seed); despertar el proyecto solo si el burn pide tier 1 |

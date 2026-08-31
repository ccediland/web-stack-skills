---
name: bitacora-web-stack-skills
title: web-stack-skills — bitácora
description: >-
  Registro append-only de lo que pasó en este repo y cuándo. Es RASTRO, no fuente: los hechos
  atemporales viven en RESIDENT.md y las reglas en CLAUDE.md; aquí solo se referencian.
last_updated: 2026-08-31
status: vigente
supersede: ninguno
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

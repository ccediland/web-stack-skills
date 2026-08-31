---
name: web-stack-skills-mapa
title: web-stack-skills — MAPA
description: Qué archivos hay en este repo y para qué sirve cada uno. Se lee para ubicarse sin barrer el repo entero. Las filas las genera `mapa.py`; la columna «Qué es» se escribe a mano.
last_updated: 2026-08-31
status: vigente
supersede: ninguno
---

# web-stack-skills — MAPA

Qué hay aquí y para qué sirve. Una fila describe **la unidad más pequeña con un
propósito distinto**: un archivo si es único, un directorio si sus archivos son
homogéneos (ahí la fila dice cuántos son y qué convención siguen).

**La columna `Autoridad` es la que hay que leer antes de editar nada:**

| Valor | Significa |
|---|---|
| `raíz` | Fuente de verdad. Se edita aquí. |
| `proyección` | Generado desde otra cosa. **No se edita aquí** — se corrige la fuente. |
| `evidencia` | Registro congelado de algo que pasó. No se actualiza. |
| `andamio` | Config, scaffold, herramienta. Se toca cuando estorba. |

<!-- MAPA:INICIO — filas generadas por mapa.py. La columna «Qué es» se escribe a mano y se conserva. -->
| Ruta | Qué es | Autoridad |
|---|---|---|
| `.claude-plugin/marketplace.json` | Declara este repo como marketplace de plugins. | andamio |
| `.gitignore` | Qué no entra. | andamio |
| `BITACORA.md` | Qué pasó y cuándo, append-only. | raíz |
| `CLAUDE.md` | Cómo tocar este repo sin romperlo. | raíz |
| `LICENSE` | Licencia. | andamio |
| `README.md` | Qué es el catálogo de stack web, para alguien de fuera. | raíz |
| `RESIDENT.md` | Cómo es el catálogo hoy, sus olas de construcción y el calendario de drift. | raíz |
| `archive/execution-plan-2026-08.md` | Plan de ejecución de agosto, ya cumplido. Archivo. | evidencia |
| `plugin/` | **El catálogo.** Las skills del stack: tokens y CSS, datos, i18n, medios, motion, SEO/AEO, seguridad, puertas de CI, WebGL, formularios, auth y el playbook que las compone. `furever-web` fue su banco de pruebas. | raíz |
| `upstream-suggestions.md` | Hallazgos que valdría subir a las fuentes originales de las recetas. | raíz |
<!-- MAPA:FIN -->

# web-stack-skills — CLAUDE.md (dev doc)

> Lee `RESIDENT.md` PRIMERO (fuente de verdad: verdictos, pins, decisiones, estado) y luego este archivo.
> Este doc solo cubre el CÓMO operar el repo; apunta al RESIDENT, nunca lo duplica.

## Qué es

Marketplace de plugins de Claude Code (público, MIT): un plugin `web-stack` que agrupa skills que codifican un stack web premium (Astro + Cloudflare Workers Static Assets). El repo guarda FUENTES de las skills — jamás se commitean los `.skill` empaquetados.

## Mapa del repo

    .claude-plugin/
      marketplace.json          un plugin (web-stack, source ".")
      plugin.json               manifiesto; array "skills" = registro explícito de cada skill
    skills/<nombre>/            una carpeta por skill (nombre de carpeta == name del frontmatter)
      SKILL.md                  veredicto + receta (frontmatter SOLO name + description)
      references/*.md           configs, plantillas, gotchas (progressive disclosure)
    deferred/                   skills sin sustancia aún — FUERA del plugin (exclusión estructural)
    RESIDENT.md                 doc vivo canónico (leer primero)
    v1-finalization-plan.md     plan mortal del sprint en curso (se archiva al ship)
    README.md · LICENSE

## Comandos

Validar y empacar con los scripts canónicos del skill-creator de Anthropic (repo `anthropics/skills`, skill `skill-creator`; en claude.ai viven en `/mnt/skills/examples/skill-creator/scripts/`):

    python quick_validate.py skills/<nombre>     # debe imprimir "Skill is valid!"
    python package_skill.py skills/<nombre>      # emite <nombre>.skill (NO se commitea)

Probar el marketplace desde Claude Code:

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Tras instalar, probar triggering: cada skill debe disparar con sus frases objetivo y NO con las de una hermana.

## Convenciones y guardrails

- Toda skill vive bajo `skills/<nombre>/` Y se registra en el array `"skills"` de `plugin.json` — el auto-discovery solo no basta. Lo diferido va en `deferred/` (fuera del plugin hasta tener sustancia; se promueve moviéndolo a `skills/` + registrándolo).
- Frontmatter de `SKILL.md` — SOLO `name` y `description`. References llevan `title` / `summary` / `last_updated` / `applies_to`.
- Reglas duras de `description` (las impone `quick_validate` y el YAML) — ≤1024 chars; sin `<` ni `>`; sin dos-puntos-espacio a media cadena (rompe YAML — usar guión largo); `name` en kebab-case ≤64 sin la palabra "claude".
- Reglas de trigger surface (disciplina propia, la superficie más frágil) — cada description termina con frase de frontera ("Not for X — use Y") que nombra a la hermana dueña; al editar una description, auditar solape de frases contra las otras (ver §8 del RESIDENT, hallazgos 2026-08-17).
- House style de bundles (md-house-style) — body solo `#`/`##`/`###`, sin bold/itálicas/HR/H4; tablas bienvenidas; TOC en archivos de más de ~100 líneas.
- Pins — cada pin duro y cada workaround por issue-number lleva review-gate (§3 y §7 del RESIDENT). No bumpear pins casualmente; los bumps van por fase de re-pin (B0) o por review-gate disparado.
- Git — trabajar SIEMPRE en rama `claude/<nombre>`, nunca en `main`; PR y esperar OK de Carlos para merge. Commits atómicos por unidad lógica.
- Al cerrar trabajo importante — reflejar decisiones/estado en `RESIDENT.md` (§9/§11) y aquí lo que cambie para el dev.

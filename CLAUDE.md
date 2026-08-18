# web-stack-skills — CLAUDE.md (dev doc)

> Lee `RESIDENT.md` PRIMERO (fuente de verdad: verdictos, pins, decisiones, estado) y luego este archivo.
> Este doc solo cubre el CÓMO operar el repo; apunta al RESIDENT, nunca lo duplica.

## Qué es

Marketplace de plugins de Claude Code (público, MIT): un plugin `web-stack` que agrupa skills que codifican un stack web premium (Astro + Cloudflare Workers Static Assets). El repo guarda FUENTES de las skills — jamás se commitean los `.skill` empaquetados.

## Mapa del repo

    .claude-plugin/
      marketplace.json          un plugin (web-stack, source "./plugin" — SOLO el subdirectorio viaja al installer)
    plugin/                     el payload instalable completo (y nada más que él)
      .claude-plugin/plugin.json  manifiesto; array "skills" = registro explícito de cada skill (rutas relativas a plugin/)
      skills/<nombre>/          una carpeta por skill (nombre de carpeta == name del frontmatter)
        SKILL.md                veredicto + receta (frontmatter SOLO name + description)
        references/*.md         configs, plantillas, gotchas (progressive disclosure)
      LICENSE                   copia — MIT exige que la licencia acompañe lo distribuido
    deferred/                   (no existe hoy — se recrea al diferir) skills sin sustancia, FUERA del plugin
    RESIDENT.md                 doc vivo canónico (leer primero); §12 = calendario de drift
    archive/                    planes cerrados (el roadmap full-catalog vive ahí desde v2.0.0)
    README.md · LICENSE

Regla del layout: TODO lo que deba llegar al consumidor vive bajo `plugin/`; los docs del repo (RESIDENT, CLAUDE, README, archive) viven FUERA. No existe mecanismo de ignore en plugins — el source dir se copia completo al cache del installer; el subdirectorio ES el filtro.

## Comandos

Validar y empacar con los scripts canónicos del skill-creator de Anthropic (repo `anthropics/skills`, skill `skill-creator`; en claude.ai viven en `/mnt/skills/examples/skill-creator/scripts/`):

    python quick_validate.py plugin/skills/<nombre>     # debe imprimir "Skill is valid!"
    python package_skill.py plugin/skills/<nombre>      # emite <nombre>.skill (NO se commitea)

Probar el marketplace desde Claude Code:

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Tras instalar, probar triggering: cada skill debe disparar con sus frases objetivo y NO con las de una hermana.

## Política de versionado (semver del PLUGIN)

- El plugin versiona el CATÁLOGO entero; las skills NO llevan semver propio (19 versiones paralelas serían drift garantizado — la versión del plugin es la única verdad).
- MAJOR — breaking del catálogo: retirar o renombrar una skill, romper el contrato de una recipe o schema que los consumidores citan, o reestructurar el layout instalable.
- MINOR — skill nueva, o extensión versionada de una existente (las absorciones de W4 — view transitions a motion, content modeling a cms — son el precedente del patrón).
- PATCH — contenido, descriptions, pins, references o fixes sin superficie nueva.
- Lockstep obligatorio: `plugin/.claude-plugin/plugin.json` y `.claude-plugin/marketplace.json` llevan SIEMPRE la misma versión; verificarlo es parte del cierre adversarial pre-merge de cada release.
- Cada versión shippeada = tag anotado `vX.Y.Z` + GitHub Release con notas.
- Cadencia de mantenimiento post-v2: el calendario de drift (RESIDENT §12) dispara re-pins; un run de drift que toque contenido shippea PATCH (o MINOR si abre superficie).

## Convenciones y guardrails

- Toda skill vive bajo `plugin/skills/<nombre>/` Y se registra en el array `"skills"` de `plugin.json` — el auto-discovery solo no basta. Lo diferido va en `deferred/` (fuera del plugin hasta tener sustancia; se promueve con `git mv` a `skills/` + registro — así se promovió la #10 en D).
- `stack-integration-playbook` es la única skill de COMPOSICIÓN: lecciones cross-cutting nuevas de builds compuestos van a sus seams/recipes; lecciones de una sola skill van a la ref de esa skill. Al cerrar cada ola, expandir el playbook (mapa de stack y recipes) con lo que la ola probó.
- Frontmatter de `SKILL.md` — SOLO `name` y `description`. References llevan `title` / `summary` / `last_updated` / `applies_to`.
- Reglas duras de `description` (las impone `quick_validate` y el YAML) — ≤1024 chars; sin `<` ni `>`; sin dos-puntos-espacio a media cadena (rompe YAML — usar guión largo); `name` en kebab-case ≤64 sin la palabra "claude".
- Reglas de trigger surface (disciplina propia, la superficie más frágil) — cada description termina con frase de frontera ("Not for X — use Y") que nombra a la hermana dueña; al editar una description, auditar solape de frases contra las otras (ver §8 del RESIDENT, hallazgos 2026-08-17).
- House style de bundles (md-house-style) — body solo `#`/`##`/`###`, sin bold/itálicas/HR/H4; tablas bienvenidas; TOC en archivos de más de ~100 líneas.
- Pins — cada pin duro y cada workaround por issue-number lleva review-gate (§3 y §7 del RESIDENT). No bumpear pins casualmente; los bumps van por fase de re-pin (B0) o por review-gate disparado.
- Git — trabajar SIEMPRE en rama `claude/<nombre>`, nunca en `main`; PR y esperar OK de Carlos para merge. Commits atómicos por unidad lógica.
- Al cerrar trabajo importante — reflejar decisiones/estado en `RESIDENT.md` (§9/§11) y aquí lo que cambie para el dev.

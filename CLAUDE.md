# web-stack-skills — CLAUDE.md (dev doc)

> Lee `RESIDENT.md` PRIMERO (fuente de verdad: qué es, stack, veredictos, decisiones) y luego este
> archivo. Este doc solo cubre el CÓMO operar el repo; apunta al RESIDENT, nunca lo duplica.

## Cómo verificar

No hay build ni tests de producto — el repo guarda fuentes Markdown de skills, no código que corra.
El verde real tiene dos capas:

    python3 ~/.claude/tools/audita.py revisa .      # el doc-set cumple el estándar (avisa, no bloquea)
    python quick_validate.py plugin/skills/<nombre>  # una skill es válida (debe imprimir "Skill is valid!")

`audita.py` corre también en el hook `pre-commit` del repo. Un aviso que aparece ahí no bloquea el
commit, pero es trabajo pendiente — nunca se ignora en silencio. `quick_validate.py` (del
skill-creator de Anthropic) es lo único que certifica que una skill instala: correrlo después de
tocar cualquier `SKILL.md` o `references/`.

Tras editar una `description`, auditar solape de triggers contra las otras 18 skills (la superficie
más frágil del catálogo — ver `docs/skill-verdicts.md` y el historial de fixes en `BITACORA.md`).

## Qué es

Marketplace de plugins de Claude Code (público, MIT): un plugin `web-stack` que agrupa skills que
codifican un stack web premium (Astro + Cloudflare Workers Static Assets). El repo guarda FUENTES de
las skills — jamás se commitean los `.skill` empaquetados.

## Mapa del repo

El inventario completo con descripciones vive en `MAPA.md`. Regla del layout: TODO lo que deba
llegar al consumidor vive bajo `plugin/`; los docs del repo (RESIDENT, CLAUDE, README, BITACORA,
archive) viven FUERA. No existe mecanismo de ignore en plugins — el source dir se copia completo al
cache del installer, el subdirectorio ES el filtro.

## Comandos

Validar y empacar con los scripts canónicos del skill-creator de Anthropic (repo `anthropics/skills`,
skill `skill-creator`; en claude.ai viven en `/mnt/skills/examples/skill-creator/scripts/`):

    python quick_validate.py plugin/skills/<nombre>     # debe imprimir "Skill is valid!"
    python package_skill.py plugin/skills/<nombre>      # emite <nombre>.skill (NO se commitea)

Probar el marketplace desde Claude Code:

    /plugin marketplace add ccediland/web-stack-skills
    /plugin install web-stack@web-stack-skills

Tras instalar, probar triggering: cada skill debe disparar con sus frases objetivo y NO con las de
una hermana.

## Política de versionado (semver del PLUGIN)

- El plugin versiona el CATÁLOGO entero; las skills NO llevan semver propio (19 versiones paralelas
  serían drift garantizado — la versión del plugin es la única verdad).
- MAJOR — breaking del catálogo: retirar o renombrar una skill, romper el contrato de una recipe o
  schema que los consumidores citan, o reestructurar el layout instalable.
- MINOR — skill nueva, o extensión versionada de una existente.
- PATCH — contenido, descriptions, pins, references o fixes sin superficie nueva.
- Lockstep obligatorio: `plugin/.claude-plugin/plugin.json` y `.claude-plugin/marketplace.json`
  llevan SIEMPRE la misma versión; verificarlo es parte del cierre adversarial pre-merge de cada
  release.
- Cada versión shippeada = tag anotado `vX.Y.Z` + GitHub Release con notas.
- Cadencia de mantenimiento post-v2: el calendario de drift (`docs/calendario-drift.md`) dispara
  re-pins; un run de drift que toque contenido shippea PATCH (o MINOR si abre superficie).

## Convenciones y guardrails

- Toda skill vive bajo `plugin/skills/<nombre>/` Y se registra en el array `"skills"` de
  `plugin.json` — el auto-discovery solo no basta. Lo diferido va en `deferred/` (fuera del plugin
  hasta tener sustancia; se promueve con `git mv` a `skills/` + registro).
- `stack-integration-playbook` es la única skill de COMPOSICIÓN: lecciones cross-cutting de builds
  compuestos van a sus seams/recipes; lecciones de una sola skill van a la ref de esa skill.
- Frontmatter de `SKILL.md` — SOLO `name` y `description`. References llevan `title` / `summary` /
  `last_updated` / `applies_to`.
- Reglas duras de `description` (las impone `quick_validate` y el YAML) — ≤1024 chars; sin `<` ni
  `>`; sin dos-puntos-espacio a media cadena (rompe YAML — usar guión largo); `name` en kebab-case
  ≤64 sin la palabra "claude".
- House style de bundles (md-house-style) — body solo `#`/`##`/`###`, sin bold/itálicas/HR/H4;
  tablas bienvenidas; TOC en archivos de más de ~100 líneas.
- Git — trabajar SIEMPRE en rama `claude/<nombre>`, nunca en `main`; PR y mergear (OK permanente).
  Commits atómicos por unidad lógica.
- Al cerrar trabajo importante — reflejar decisiones/estado nuevo en `RESIDENT.md` §7 y una entrada
  en `BITACORA.md` (ver abajo).

## Bitácora

Al cerrar cualquier sesión que haya tocado este repo, **agrega su entrada al final de
`BITACORA.md` sin preguntar y sin anunciarlo**. Una sesión sin entrada no cerró.

El formato está en la cabecera de ese archivo. Es **append-only**: nunca se edita ni se reordena
una entrada pasada, y una revisión posterior va abajo con su propia fecha — jamás como nota en la
cabecera.

Qué va dónde, con la prueba mecánica del estándar:

> ¿Lleva una **fecha**? → `BITACORA.md`.
> ¿Sigue siendo verdad en tres meses sin que nadie lo toque? → `RESIDENT.md`.
> ¿Es sobre cómo tocar los archivos? → aquí.

Esto no es burocracia: es lo que mantiene a `RESIDENT.md` atemporal. Un RESIDENT que carga la
narrativa de cada sesión cambia todos los días, y entonces el knowledge de su Project queda
desfasado siempre.

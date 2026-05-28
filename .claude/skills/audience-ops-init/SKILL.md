---
name: audience-ops-init
description: "Inicializa una instancia de Audience Ops en el directorio actual creando una carpeta `audience-ops/` con la estructura completa (strategy, voice, channels, ideas, publications). Una instancia por repo."
metadata:
  author: r-bart
  version: "0.3.0"
---

# init — Bootstrap de una instancia de Audience Ops

## Cuándo usar esta skill

- El directorio actual no tiene aún una carpeta `audience-ops/`.
- Quieres montar la operativa de contenidos para este proyecto (sea un repo de código o un directorio dedicado).

Una instancia por repo. Si necesitas operar varios proyectos en paralelo, instala el tool en cada repo y `init` en cada uno.

## Entradas

- Ninguna obligatoria. Todo se pregunta en el flujo.

## Lectura previa

Antes de actuar, leer:

1. `./audience-ops/config.yaml` — comportamiento depende de la invocación:
   - **Sin flag**: si existe ya, ABORT con mensaje: "audience-ops/ ya está inicializado en este directorio. Para editar, modifica los ficheros directamente. Para empezar de cero, `rm -rf audience-ops/` antes de re-invocar. Para añadir proyectos a la instancia existente, invoca con `--add-project`."
   - **Con `--add-project`**: si existe ya, entrar en sub-modo add-project (saltar todos los pasos excepto el nuevo Paso 7 · Proyectos y el resumen final). Si NO existe ya, error con mensaje: "No hay instancia de audience-ops en este directorio. Ejecuta `/audience-ops-init` sin flags primero."
2. `./portfolio.yaml`, `./projects/` — si existen (residuos del modelo pre-single-instance): avisar al usuario una vez ("detectado layout legacy; ver release notes de v0.2.0 para migración manual") y continuar con la creación de `audience-ops/` en paralelo.

## Pasos

### Paso 1 · Confirmar contexto

Mostrar al usuario:

```
Voy a crear `./audience-ops/` con:
  config.yaml, strategy.md, voice.md, channels/, ideas/, publications/

El "proyecto" es implícitamente este directorio: <basename(CWD)>.

¿Continuar?
```

Si el usuario confirma, seguir. Si no, abortar.

### Paso 2 · Crear estructura base

Crear:

```
./audience-ops/
├── config.yaml          (con defaults; placeholder en owner)
├── strategy.md          (placeholder, se rellena en paso 4)
├── voice.md             (placeholder, se rellena en paso 5)
├── channels/            (se rellena en paso 6)
│   └── .gitkeep
├── ideas/
│   └── _inbox.md        (vacío con cabecera)
└── publications/
    └── .gitkeep
```

Contenido de `audience-ops/config.yaml`:

```yaml
# config.yaml — cómo se comporta esta instancia de Audience Ops

owner:
  name:
  email:

behavior:
  inbox_auto_promote: false
  cleanup_threshold_days: 90
  stale_draft_days: 30
  stale_inbox_days: 30
  strategy_review_months: 4
  paused_archive_threshold_months: 6
  learnings_prompt_threshold_days: 60
  prompt_learnings_on_publish: true
  scheduler: typefully
```

Contenido de `audience-ops/ideas/_inbox.md`:

```markdown
# Inbox

<!-- Una línea por idea. Formato: YYYY-MM-DD · texto -->
```

### Paso 3 · Owner (opcional)

Si `audience-ops/config.yaml` `owner.name` o `owner.email` están vacíos, preguntar al usuario y rellenar. El usuario puede saltar dejándolos en blanco; no se vuelve a preguntar en el futuro.

### Paso 4 · Sub-flujo · Estrategia

Si existe la skill `audience-ops-strategy` (`.claude/skills/audience-ops-strategy/SKILL.md`), delegar la creación de `audience-ops/strategy.md` a esa skill.

Si no existe (caso bootstrap muy temprano), hacer una versión rudimentaria inline. Preguntar al usuario por:

- Posicionamiento (frase)
- Audiencia / ICP (rol + trigger + qué probó + dónde lee)
- Pilares (2-5; slug + descripción)
- Objetivos (opcional; formato `<periodo>: <métrica>`)
- Anti-temas (opcional)

Y construir `audience-ops/strategy.md` con frontmatter (`last_reviewed: <hoy>`, `pillars: [...]`) y prosa con H2s. **Sin campo `slug` en frontmatter** (el "proyecto" es el repo host).

Si el usuario quiere posponer alguna sección, dejarla con `_pendiente_` y avisar.

### Paso 5 · Sub-flujo · Voz y tono

Antes de preguntar, mostrar al usuario los **ejemplos canónicos** etiquetados como "Ejemplo (no obligatorio)" para anclar expectativas. El usuario puede:

- Decir "como el ejemplo" en cualquier sub-sección → el skill escribe el ejemplo verbatim.
- Refinarlo o reescribirlo a su gusto.
- Dejarlo en `_pendiente_` y volver más tarde editando `voice.md` a mano.

**Ejemplos canónicos**:

```markdown
## Atributos
- directo, técnico, sin hype
- primera persona honesta

## Sí
- datos concretos, ejemplos del propio uso, decisiones explicadas
- digresiones cortas si son honestas

## No
- jerga marketinera, superlatives
- listas "10 razones por las que..."
- preguntas retóricas como hook

## Referencias
- escribe como Paul Graham, no como Gary Vee
```

Preguntar al usuario por:

- **Atributos** (3–5 adjetivos de la voz, o di "como el ejemplo").
- **Sí** (3–5 cosas que sí se hacen, o di "como el ejemplo").
- **No** (3–5 cosas que no se hacen, o di "como el ejemplo").
- **Referencias** (autores/marcas que inspiran la voz, opcional, o di "como el ejemplo").

Escribir `audience-ops/voice.md`:

```markdown
## Atributos
- <attr-1>
- ...

## Sí
- <si-1>
- ...

## No
- <no-1>
- ...

## Referencias
- <ref-1>
```

### Paso 6 · Sub-flujo · Canales

Preguntar: "¿Qué canales activos quieres registrar para este proyecto?" (ejemplos: X, LinkedIn, blog, newsletter, YouTube, Reddit).

Para cada canal indicado, preguntar:

- **id** (`x`, `linkedin`, `newsletter`, etc.)
- **handle** (`@usuario`, URL, o nombre humano)
- **url** (opcional)
- **cadence** (ej. `3/week`, `1/month`)
- **platform** (opcional, ej. `beehiiv`, `ghost` para newsletters)

Crear `audience-ops/channels/<id>.md` por cada uno:

```markdown
---
id: <id>
handle: "<handle>"
url: <url>            # omitir si no se dio
platform: <platform>  # omitir si no aplica
cadence: <cadence>
status: active
---

## Formato

<formato canónico para este canal; rellenar con sugerencias según el id>

## Ajustes de voz (override sobre voice.md)

<vacío inicialmente; el usuario lo puede rellenar después>

## Sí

<sugerencias>

## No

<sugerencias>
```

Eliminar `audience-ops/channels/.gitkeep` si se creó al menos un canal.

#### Sugerencias por defecto de formato según `id`

- **x**: Threads de 5–7 tweets; primer tweet = hook concreto; una idea por tweet; sin numerar.
- **linkedin**: Posts de 1200–1500 caracteres; primera línea = hook; párrafos cortos.
- **blog**: Long-form. Estructura: hook → tesis → desarrollo → cierre. 800–1500 palabras.
- **newsletter**: Subject ≤50 chars sin clickbait; preheader complementario; hook (2-3 frases) → cuerpo (1 idea) → CTA opcional; 400–700 palabras. **Importante**: requiere campos `subject` y `preheader` en frontmatter de cada publicación.
- **youtube**: Hook en los primeros 5s; estructura clara; CTA al final.
- **reddit**: Adaptarse al subreddit; sin auto-promoción explícita; valor primero.

Si el `id` no está en la lista, dejar la sección `## Formato` con un placeholder y avisar al usuario que la complete.

### Paso 7 · Sub-flujo · Proyectos (opcional, v0.3.0+)

Si la instancia va a alojar contenido para varios productos (o uno solo con dossier explícito), declarar proyectos aquí crea los scaffolds en `audience-ops/projects/`. Estos dossiers son **opcionales** — saltarlos no rompe nada; pueden añadirse después con `/audience-ops-init --add-project`.

Preguntar al usuario:

> "¿Quieres declarar proyectos ahora? Útil si vas a usar esta instancia para varios productos (ej. brakinglab, verxion, blog personal). Saltable; añadibles después con `--add-project`."

Si **declina**: omitir el paso. No crear `audience-ops/projects/`.

Si **acepta**, para cada proyecto que el usuario nombre, preguntar (con sugerencias):

- **Name**: nombre humano (ej. "Brakinglab").
- **Slug prefix**: 2–3 chars que prefijaran los slugs de ideas vinculadas. Sugerir derivado del nombre (`Brakinglab` → `bl`; `Verxion` → `vx`; `Audience Ops` → `aop`). Validar único contra dossiers ya creados en esta invocación. Si colisiona, pedir alternativa.
- **URL** (opcional): URL principal del proyecto.
- **Descripción corta** (opcional): una frase que va al `## Qué es`. Si no, dejarlo vacío.

Crear `audience-ops/projects/<name-kebab-case>.md` por cada proyecto:

```markdown
---
name: <name>
slug_prefix: <prefix>
url: <url>                              # omitir si no se dio
status: building                        # default — el usuario lo edita después
last_updated: <YYYY-MM-DD de hoy>
---

## Qué es

<descripción si se dio; vacío si no>

## Audiencia

<!-- Quién se beneficia, casos de uso reales. Rellénalo cuando lo tengas claro. -->

## Story log

<!-- Reverse-chronological. Una línea por evento publicable (decisión, métrica, bug, conversación). -->

## Ángulos abiertos

<!-- Material del story log que podría madurar a idea estructurada. -->
```

Continuar preguntando "¿otro proyecto?" hasta que el usuario diga no.

Si **se está en modo `--add-project`** (instancia existente), tras crear los nuevos dossiers, saltar directo al Paso 8 (Resumen final) y reportar solo los dossiers añadidos esta sesión.

**Edge cases del Paso 7**:
- `audience-ops/projects/` ya existe con ficheros (puede pasar en `--add-project`): leer los `slug_prefix:` ya declarados y validar que los nuevos no colisionen.
- Slug-prefix sugerido colisiona con uno existente: pedir alternativa, sugerir con un sufijo numérico (`bl2`, `bl3`) o sufijo letra (`bla`).
- Nombre de proyecto en kebab-case colisiona con un fichero existente en `audience-ops/projects/`: pedir desambiguación (`brakinglab-v2.md`, etc.).
- Usuario interrumpe a mitad de declarar el N-ésimo proyecto: los dossiers ya escritos quedan; los pendientes se omiten.

### Paso 8 · Resumen final

Mostrar al usuario:

- Resumen de lo creado: estructura de `audience-ops/`, canales registrados.
- El "proyecto" del repo es `<basename(CWD)>`.
- Si se declararon proyectos en Paso 7: lista de dossiers creados con sus `slug_prefix`. Ej.:
  - `audience-ops/projects/brakinglab.md` (prefix `bl`)
  - `audience-ops/projects/verxion.md` (prefix `vx`)
- Para los dossiers: rellenar `## Qué es` y `## Audiencia` a mano cuando tengas un momento; alimentar `## Story log` cuando pasen cosas publicables (decisiones, métricas, bugs).
- Para añadir un proyecto después: `/audience-ops-init --add-project`.
- Siguientes pasos:
  - "Captura tu primera idea: `/audience-ops-idea \"<texto>\"`".
  - "Cuando tengas 3-5 ideas, genera un primer draft: `/audience-ops-draft <idea-slug> <channel>`".
  - "Revisa la estrategia en cualquier momento: `/audience-ops-strategy`".

En **modo `--add-project`**, el resumen lista solo los dossiers nuevos creados esta sesión (no la instancia entera).

## Escritura · Ficheros creados o modificados

| Fichero | Acción |
|---|---|
| `./audience-ops/config.yaml` | Crear |
| `./audience-ops/strategy.md` | Crear |
| `./audience-ops/voice.md` | Crear |
| `./audience-ops/channels/<id>.md` | Crear uno por canal |
| `./audience-ops/ideas/_inbox.md` | Crear con cabecera |
| `./audience-ops/publications/.gitkeep` | Crear |
| `./audience-ops/projects/<project-slug>.md` | Crear uno por proyecto declarado (opcional, Paso 7) |

## Criterios de éxito

- `./audience-ops/` existe con `config.yaml`, `strategy.md`, `voice.md`, `channels/<id>.md` (≥1), `ideas/_inbox.md`, `publications/`.
- `strategy.md` tiene frontmatter con `pillars` listados y `last_reviewed: hoy`. **Sin** `slug:`.
- Cada `channels/<id>.md` tiene frontmatter completo (`id`, `handle`, `cadence`, `status`).
- El usuario confirmó cada paso de escritura.
- `portfolio.yaml` no fue creado (no existe).
- `projects/<slug>/` no fue creado.
- Si se declararon proyectos en Paso 7, los dossiers existen en `audience-ops/projects/` con frontmatter completo (`name`, `slug_prefix`, `last_updated`) y las 4 secciones canónicas.
- En modo `--add-project`: solo se crearon los dossiers nuevos; el resto de la instancia quedó intacta.

## Errores y casos límite

- **`./audience-ops/` ya existe con config.yaml**: abortar con mensaje claro. No sobreescribir.
- **Layout legacy detectado** (presencia de `./portfolio.yaml` o `./projects/` — del modelo pre-single-instance): avisar una vez, continuar creando `audience-ops/` en paralelo. La migración manual es decisión del usuario.
- **Usuario interrumpe a mitad** (por ejemplo, tras crear estructura pero antes de canales): dejar lo creado, avisar de qué falta, sugerir reanudar invocando `init` (que detectará `audience-ops/` parcial y abortará — el usuario completa a mano).
- **Channel `id` desconocido**: aceptar, placeholder en formato + warning.
- **Usuario no quiere voz aún**: crear `voice.md` con secciones marcadas `_pendiente_`.
- **`--add-project` invocado sin instancia existente**: error con mensaje "no audience-ops/ found; run `/audience-ops-init` first".
- **Slug-prefix colisiona** con dossier existente: re-prompt al usuario con sugerencia (`bl2`, `bla`, etc.).
- **Project name (kebab-case) colisiona** con dossier existente: pedir disambiguation (`brakinglab-v2`, etc.).

## Principios que aplica

- **Cero magia.** Confirmación antes de cada escritura.
- **Convención sobre configuración.** `audience-ops/` es el namespace fijo; el "proyecto" se deriva del repo host.
- **Una sola fuente de verdad.** Owner en config.yaml, no duplicado en portfolio (que no existe).
- **Frontmatter mínimo.** Solo `pillars` y `last_reviewed` en strategy. Sin `slug`.
- **Single-instance.** Una instancia por repo. Multi-proyecto = varios repos.

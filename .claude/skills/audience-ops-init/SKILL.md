---
name: audience-ops-init
description: "Inicializa una instancia de Audience Ops en el directorio actual creando una carpeta `audience-ops/` con la estructura completa (strategy, voice, channels, ideas, publications). Una instancia por repo."
metadata:
  author: r-bart
  version: "0.2.0"
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

1. `./audience-ops/config.yaml` — si existe ya, ABORT con mensaje: "audience-ops/ ya está inicializado en este directorio. Para editar, modifica los ficheros directamente. Para empezar de cero, `rm -rf audience-ops/` antes de re-invocar."
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

### Paso 7 · Resumen final

Mostrar al usuario:

- Resumen de lo creado: estructura de `audience-ops/`, canales registrados.
- El "proyecto" del repo es `<basename(CWD)>`.
- Siguientes pasos:
  - "Captura tu primera idea: `/audience-ops-idea \"<texto>\"`".
  - "Cuando tengas 3-5 ideas, genera un primer draft: `/audience-ops-draft <idea-slug> <channel>`".
  - "Revisa la estrategia en cualquier momento: `/audience-ops-strategy`".

## Escritura · Ficheros creados o modificados

| Fichero | Acción |
|---|---|
| `./audience-ops/config.yaml` | Crear |
| `./audience-ops/strategy.md` | Crear |
| `./audience-ops/voice.md` | Crear |
| `./audience-ops/channels/<id>.md` | Crear uno por canal |
| `./audience-ops/ideas/_inbox.md` | Crear con cabecera |
| `./audience-ops/publications/.gitkeep` | Crear |

## Criterios de éxito

- `./audience-ops/` existe con `config.yaml`, `strategy.md`, `voice.md`, `channels/<id>.md` (≥1), `ideas/_inbox.md`, `publications/`.
- `strategy.md` tiene frontmatter con `pillars` listados y `last_reviewed: hoy`. **Sin** `slug:`.
- Cada `channels/<id>.md` tiene frontmatter completo (`id`, `handle`, `cadence`, `status`).
- El usuario confirmó cada paso de escritura.
- `portfolio.yaml` no fue creado (no existe).
- `projects/<slug>/` no fue creado.

## Errores y casos límite

- **`./audience-ops/` ya existe con config.yaml**: abortar con mensaje claro. No sobreescribir.
- **Layout legacy detectado** (presencia de `./portfolio.yaml` o `./projects/` — del modelo pre-single-instance): avisar una vez, continuar creando `audience-ops/` en paralelo. La migración manual es decisión del usuario.
- **Usuario interrumpe a mitad** (por ejemplo, tras crear estructura pero antes de canales): dejar lo creado, avisar de qué falta, sugerir reanudar invocando `init` (que detectará `audience-ops/` parcial y abortará — el usuario completa a mano).
- **Channel `id` desconocido**: aceptar, placeholder en formato + warning.
- **Usuario no quiere voz aún**: crear `voice.md` con secciones marcadas `_pendiente_`.

## Principios que aplica

- **Cero magia.** Confirmación antes de cada escritura.
- **Convención sobre configuración.** `audience-ops/` es el namespace fijo; el "proyecto" se deriva del repo host.
- **Una sola fuente de verdad.** Owner en config.yaml, no duplicado en portfolio (que no existe).
- **Frontmatter mínimo.** Solo `pillars` y `last_reviewed` en strategy. Sin `slug`.
- **Single-instance.** Una instancia por repo. Multi-proyecto = varios repos.

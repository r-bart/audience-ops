---
name: audience-ops-init
description: "Bootstrap del repo Audience Ops (si está vacío) o añade un nuevo proyecto. Encadena la creación de estructura, estrategia, voz y canales en un flujo guiado."
metadata:
  author: r-bart
  version: "0.10.1"
---

# init — Bootstrap del repo + primer proyecto

## Cuándo usar esta skill

- El repo es nuevo y `portfolio.yaml` está vacío (sin proyectos).
- Quieres añadir un nuevo proyecto a un repo Audience Ops ya inicializado.

## Entradas

- Ninguna obligatoria. Todo se pregunta en el flujo.
- Opcional: el usuario puede pasar un slug propuesto o un name al invocar.

## Lectura previa

Antes de actuar, leer:

1. `portfolio.yaml` — para saber si hay owner definido y qué proyectos existen.
2. `config.yaml` — para ver defaults actuales.
3. `projects/` — listar slugs ya existentes para evitar colisiones.

## Pasos

### Paso 1 · Detectar estado del repo

- Si `portfolio.yaml` no tiene `owner:` definido → **modo bootstrap completo** (paso 2 incluido).
- Si `owner:` está definido y `projects:` está vacío o tiene entradas → **modo añadir proyecto** (saltar al paso 3).

### Paso 2 · Bootstrap (solo si modo bootstrap completo)

Preguntar al usuario:

- Su nombre (para `owner:` en `portfolio.yaml`).
- Su email (opcional, para `config.yaml`).

Escribir en `portfolio.yaml`:

```yaml
owner: <nombre>
projects: []
```

Escribir en `config.yaml` (solo si los campos `owner.name` / `owner.email` están vacíos):

```yaml
owner:
  name: <nombre>
  email: <email>
```

Confirmar al usuario antes de escribir.

### Paso 3 · Datos del nuevo proyecto

Preguntar:

- **Slug**: kebab-case, único. Validar contra `projects/` y `portfolio.yaml`. Si colisiona, pedir otro.
- **Name**: nombre humano (ej. "Verxion").
- **One-liner**: descripción de una línea.
- **Status**: `active | building | paused | archived` (default: `building`).
- **URL** (opcional): URL principal del proyecto.
- **Started** (opcional): mes/año de arranque, formato `YYYY-MM`.

### Paso 4 · Crear estructura del proyecto

Mostrar al usuario el árbol que se va a crear y pedir confirmación antes de escribir nada:

```
projects/<slug>/
├── strategy.md          (placeholder, se rellena en paso 5)
├── voice.md             (placeholder, se rellena en paso 6)
├── channels/            (se rellena en paso 7)
├── ideas/
│   └── _inbox.md        (vacío, con cabecera)
└── publications/        (vacía)
```

Tras confirmación, crear:

- `projects/<slug>/ideas/_inbox.md` con contenido:
  ```markdown
  # Inbox

  <!-- Una línea por idea. Formato: YYYY-MM-DD · texto -->
  ```
- `projects/<slug>/publications/.gitkeep` vacío.
- `projects/<slug>/channels/.gitkeep` vacío (se sustituye al definir canales).

### Paso 5 · Actualizar `portfolio.yaml`

Añadir entrada del proyecto a la lista `projects`:

```yaml
- slug: <slug>
  name: <name>
  one_liner: "<one_liner>"
  status: <status>
  url: <url>            # omitir si no se dio
  started: <started>    # omitir si no se dio
```

Si `defaults.project` en `config.yaml` está vacío y este es el primer proyecto, ofrecer fijarlo como default.

### Paso 6 · Sub-flujo · Estrategia

Si existe la skill `audience-ops-strategy` (`.claude/skills/audience-ops-strategy/SKILL.md`), delegar la creación de `strategy.md` a esa skill invocándola sobre el nuevo proyecto.

Si no existe (caso bootstrap muy temprano), hacer una versión rudimentaria inline:

Preguntar al usuario y construir `projects/<slug>/strategy.md`:

```markdown
---
slug: <slug>
last_reviewed: <YYYY-MM-DD de hoy>
pillars:
  - <pillar-1>
  - <pillar-2>
  - <pillar-3>
---

## Posicionamiento

<respuesta del usuario>

## Audiencia / ICP

<respuesta del usuario>

## Pilares

### <pillar-1>
<descripción>

### <pillar-2>
<descripción>

### <pillar-3>
<descripción>

## Objetivos

- <objetivo trimestral si lo dijo>

## Anti-temas

- <lo que no se trata, si lo dijo>
```

Si el usuario quiere posponer alguna sección, dejarla con `_pendiente_` y avisar.

### Paso 7 · Sub-flujo · Voz y tono

Preguntar al usuario por:

- **Atributos** (3–5 adjetivos de la voz).
- **Sí** (3–5 cosas que sí se hacen).
- **No** (3–5 cosas que no se hacen).
- **Referencias** (autores/marcas que inspiran la voz, opcional).

Escribir `projects/<slug>/voice.md`:

```markdown
## Atributos
- <attr-1>
- <attr-2>
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

### Paso 8 · Sub-flujo · Canales

Preguntar: "¿Qué canales activos quieres registrar para este proyecto?" (ejemplos: X, LinkedIn, blog, newsletter, YouTube, Reddit).

Para cada canal indicado, preguntar:

- **id** (slug del canal: `x`, `linkedin`, `newsletter`, etc.).
- **handle** (`@usuario`, URL, o nombre humano).
- **url** (opcional).
- **cadence** (ej. `3/week`, `1/month`).
- **platform** (opcional, ej. `beehiiv`, `ghost` para newsletters).

Crear `projects/<slug>/channels/<id>.md` para cada uno:

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

<formato canónico para este canal; se rellena con sugerencias por defecto según el id>

## Ajustes de voz (override sobre voice.md)

<vacío inicialmente; el usuario lo puede rellenar después>

## Sí

<sugerencias>

## No

<sugerencias>
```

Eliminar `projects/<slug>/channels/.gitkeep` si se creó al menos un canal.

#### Sugerencias por defecto de formato según `id`

- **x**: Threads de 5–7 tweets; primer tweet = hook concreto; una idea por tweet; sin numerar.
- **linkedin**: Posts de 1200–1500 caracteres; primera línea = hook; párrafos cortos.
- **blog**: Long-form. Estructura: hook → tesis → desarrollo → cierre. 800–1500 palabras.
- **newsletter**: Subject ≤50 chars sin clickbait; preheader complementario; hook (2-3 frases) → cuerpo (1 idea) → CTA opcional; 400–700 palabras. **Importante**: requiere campos `subject` y `preheader` en frontmatter de cada publicación.
- **youtube**: Hook en los primeros 5s; estructura clara; CTA al final.
- **reddit**: Adaptarse al subreddit; sin auto-promoción explícita; valor primero.

Si el `id` no está en la lista, dejar la sección `## Formato` con un placeholder y avisar al usuario que la complete.

### Paso 9 · Resumen final

Mostrar al usuario:

- Resumen de lo creado: ficheros, carpetas, canales registrados.
- Siguientes pasos sugeridos:
  - "Captura tu primera idea: `/audience-ops-idea \"<texto>\"`".
  - "Cuando tengas 3–5 ideas, genera un primer draft: `/audience-ops-draft <idea-slug> <channel>`".
  - "Revisa o profundiza la estrategia en cualquier momento: `/audience-ops-strategy`".

## Escritura · Ficheros creados o modificados

| Fichero | Acción |
|---|---|
| `portfolio.yaml` | Añadir owner (si bootstrap) y entrada del proyecto |
| `config.yaml` | Rellenar owner y opcionalmente `defaults.project` |
| `projects/<slug>/strategy.md` | Crear |
| `projects/<slug>/voice.md` | Crear |
| `projects/<slug>/channels/<id>.md` | Crear uno por canal |
| `projects/<slug>/ideas/_inbox.md` | Crear vacío con cabecera |
| `projects/<slug>/publications/.gitkeep` | Crear |

## Criterios de éxito

- `portfolio.yaml` contiene el nuevo proyecto en `projects:`.
- `projects/<slug>/` existe con `strategy.md`, `voice.md`, `channels/<id>.md` (≥1), `ideas/_inbox.md`, `publications/`.
- `strategy.md` tiene frontmatter con `pillars` listados y `last_reviewed` = hoy.
- Cada `channels/<id>.md` tiene frontmatter completo (`id`, `handle`, `cadence`, `status`).
- El usuario confirmó cada paso de escritura.

## Errores y casos límite

- **Slug colisiona** con proyecto existente: pedir uno nuevo, no asumir.
- **Usuario interrumpe a mitad** (por ejemplo, tras crear estructura pero antes de canales): dejar lo creado, avisar de qué falta, sugerir reanudar invocando `init` con el slug existente (modo "completar proyecto a medio hacer").
- **`portfolio.yaml` ya tiene entrada para el slug pero `projects/<slug>/` no existe**: avisar de inconsistencia, ofrecer crear la carpeta o eliminar la entrada huérfana.
- **Channel `id` desconocido**: aceptar (puede ser un canal nuevo), pero placeholder en formato + warning.
- **El usuario no quiere voz aún**: crear `voice.md` con todas las secciones marcadas `_pendiente_`. La skill `draft` tratará `_pendiente_` como "sin restricciones específicas".

## Principios que aplica

- **Cero magia.** Confirmación antes de cada escritura.
- **El proyecto se deriva del path.** Slug del proyecto = nombre de carpeta = `slug:` en yaml. Misma fuente.
- **Frontmatter mínimo.** Solo `pillars` y `last_reviewed` en strategy; resto en prosa.
- **Soft archive desde el inicio.** No se crea `archive/` por adelantado, pero la convención existe.

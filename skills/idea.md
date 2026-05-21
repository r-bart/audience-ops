---
name: audience-ops:idea
description: Captura ideas rápido al inbox de un proyecto, o promueve una entrada del inbox a idea estructurada con pilar, canales y ángulo.
---

# idea — Captura y promoción de ideas

## Cuándo usar esta skill

- **Modo quick**: el usuario tiene una idea suelta y quiere guardarla sin fricción. Se añade una línea al `_inbox.md` del proyecto.
- **Modo promote**: una entrada del inbox (o un texto formal) merece convertirse en idea estructurada con pilar, canales destino, ángulo y notas. Se crea un fichero en `ideas/`.

Una sola skill cubre ambos modos.

## Entradas

### Modo quick
- Un texto corto (el contenido de la idea).
- Opcional: slug del proyecto destino. Si no se da, se usa `defaults.project` de `config.yaml`. Si tampoco, preguntar.

### Modo promote
- O bien la línea exacta del inbox que se quiere promover, o un texto/concepto formal.
- Opcional: slug del proyecto.

## Lectura previa

1. `config.yaml` — para `defaults.project`.
2. `portfolio.yaml` — para validar que el proyecto existe.
3. `projects/<slug>/strategy.md` — para extraer los pilares válidos del proyecto y sugerir al usuario.
4. `projects/<slug>/channels/` — para conocer los canales activos del proyecto.
5. `projects/<slug>/ideas/_inbox.md` — para el modo promote (encontrar la línea origen) o quick (saber cómo va a quedar).

## Detección de modo

- Si la invocación incluye un texto sin más → modo **quick**.
- Si la invocación incluye `--promote` o pide explícitamente "promueve esto" → modo **promote**.
- Si el texto coincide con una línea existente del inbox → asumir modo **promote** y confirmar con el usuario antes de actuar.

Cuando hay duda, **preguntar** ("¿Captura rápida o promover a idea estructurada?").

## Pasos · Modo quick

### Paso 1 · Determinar proyecto

- Si el usuario indicó proyecto → usar.
- Si no, usar `defaults.project` de `config.yaml`.
- Si no hay default → preguntar, mostrando la lista de proyectos activos de `portfolio.yaml`.

### Paso 2 · Formar línea

Construir la línea con la fecha de hoy en formato `YYYY-MM-DD`:

```
2026-05-21 · <texto del usuario>
```

### Paso 3 · Append a `_inbox.md`

Añadir al final de `projects/<slug>/ideas/_inbox.md`. **Append, nunca rewrite**.

Si el fichero no existe (debería existir si el proyecto fue creado por `init`), crearlo con la cabecera estándar.

### Paso 4 · Confirmar al usuario

Mostrar la línea añadida + ruta del fichero. Sin más.

## Pasos · Modo promote

### Paso 1 · Determinar proyecto

Igual que en modo quick.

### Paso 2 · Identificar origen

- Si el usuario pasó una línea del inbox: localizarla en `_inbox.md`.
- Si pasó un texto libre: tratarlo como ángulo de partida.

### Paso 3 · Interview de estructura

Preguntar (en este orden, con sugerencias cuando se pueda):

- **Slug** (kebab-case, único dentro de `projects/<slug>/ideas/`). Sugerir basado en el texto.
- **Pilar**: mostrar la lista de pilares de `strategy.md` y pedir uno. Si el usuario quiere uno fuera de la lista, advertir y permitir.
- **Canales destino**: mostrar la lista de canales activos del proyecto; pedir cero o más. Pueden añadirse después.
- **Hook / ángulo**: una o dos frases. Cómo se va a abordar la idea.
- **Notas / research**: opcional, contenido libre.

### Paso 4 · Crear `ideas/<slug>.md`

```markdown
---
slug: <slug>
pillar: <pillar>
channels: [<channel-1>, <channel-2>]
created: <YYYY-MM-DD de hoy>
---

## Hook / ángulo

<hook>

## Notas / research

<notas>

## Referencias

<vacío inicialmente; usar `[[other-idea-slug]]` para backlinks>
```

Si el usuario no rellena alguna sección, dejarla vacía pero presente.

### Paso 5 · Marcar el origen en el inbox (si aplica)

Si la idea venía de una línea del inbox, **modificar esa línea** en `_inbox.md` añadiendo un sufijo de promoción:

Antes:
```
2026-05-19 · Comparar Garmin vs Whoop para zona 2
```

Después:
```
2026-05-19 · Comparar Garmin vs Whoop para zona 2  → ideas/garmin-vs-whoop-z2.md
```

No borrar la línea. Trazabilidad.

### Paso 6 · Confirmar al usuario

Mostrar:
- Ruta del fichero creado.
- Pilar asignado.
- Canales destino previstos.
- Recordatorio: "Cuando quieras draftear: `/audience-ops:draft <slug> <channel>`".

## Escritura · Ficheros creados o modificados

| Fichero | Modo | Acción |
|---|---|---|
| `projects/<slug>/ideas/_inbox.md` | quick | Append una línea |
| `projects/<slug>/ideas/_inbox.md` | promote | Modificar la línea origen (añadir sufijo) |
| `projects/<slug>/ideas/<idea-slug>.md` | promote | Crear |

## Criterios de éxito

### Modo quick
- `_inbox.md` tiene una línea nueva al final con prefijo `YYYY-MM-DD ·`.
- Ninguna otra línea fue modificada.

### Modo promote
- `ideas/<slug>.md` existe con frontmatter (`slug`, `pillar`, `channels`, `created`).
- Si vino de inbox, la línea origen lleva sufijo `→ ideas/<slug>.md`.
- Si el pilar no estaba en `strategy.md`, el usuario fue avisado y confirmó.

## Errores y casos límite

- **Slug colisiona** con idea existente en el proyecto: pedir otro. Si el usuario insiste, ofrecer sufijo `-2`, `-3`.
- **No hay proyecto** y no se puede determinar: pedir al usuario que ejecute `init` primero o especifique uno.
- **`strategy.md` no existe** o no tiene pilares: permitir promote sin pilar (deja el campo vacío) y avisar de que se debería completar la estrategia.
- **No hay canales activos** para sugerir: permitir promote sin canales; pueden añadirse después editando el fichero.
- **Línea de inbox ya promovida** (ya tiene sufijo `→`): avisar y preguntar si se quiere crear una promoción adicional o abortar.
- **Texto de quick contiene saltos de línea**: convertirlos a un solo espacio. Una línea = una idea.

## Principios que aplica

- **Cero magia.** Las modificaciones del inbox (append, sufijo) se muestran como diff antes de aplicar si hay duda.
- **El proyecto se deriva del path.** El fichero acaba en `projects/<slug>/ideas/`, no se duplica `project:` en el frontmatter.
- **Append-only en el inbox** para captura rápida; en promote, edición controlada de una sola línea.
- **Frontmatter mínimo.** Sin campo `status` en la idea (se deriva del estado de sus publicaciones).

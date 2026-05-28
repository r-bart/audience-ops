---
name: audience-ops-idea
description: "Captura ideas rápido al inbox de la instancia, o promueve una entrada del inbox a idea estructurada con pilar, canales y ángulo."
metadata:
  author: r-bart
  version: "0.2.0"
---

# idea — Captura y promoción de ideas

## Cuándo usar esta skill

- **Modo quick**: el usuario tiene una idea suelta y quiere guardarla sin fricción. Se añade una línea al `audience-ops/ideas/_inbox.md` de la instancia.
- **Modo promote**: una entrada del inbox (o un texto formal) merece convertirse en idea estructurada con pilar, canales destino, ángulo y notas. Se crea un fichero en `ideas/`.

Una sola skill cubre ambos modos.

El "proyecto" es implícitamente la instancia local (`./audience-ops/`). Una instancia por repo; no hay slug ni selección de proyecto.

## Entradas

### Modo quick
- Un texto corto (el contenido de la idea).

### Modo promote
- O bien la línea exacta del inbox que se quiere promover, o un texto/concepto formal.

## Lectura previa

1. `audience-ops/config.yaml` — para flags de behavior.
2. `audience-ops/strategy.md` — para extraer los pilares válidos y sugerir al usuario.
3. `audience-ops/channels/` — para conocer los canales activos.
4. `audience-ops/ideas/_inbox.md` — para el modo promote (encontrar la línea origen) o quick (saber cómo va a quedar).

Si `audience-ops/` no existe, abortar y sugerir `/audience-ops-init`.

## Detección de modo

- Si la invocación incluye un texto sin más → modo **quick**.
- Si la invocación incluye `--promote` o pide explícitamente "promueve esto" → modo **promote**.
- Si el texto coincide con una línea existente del inbox → asumir modo **promote** y confirmar con el usuario antes de actuar.

Cuando hay duda, **preguntar** ("¿Captura rápida o promover a idea estructurada?").

## Pasos · Modo quick

### Paso 1 · Formar línea

Construir la línea con la fecha de hoy en formato `YYYY-MM-DD`:

```
2026-05-21 · <texto del usuario>
```

### Paso 2 · Append a `_inbox.md`

Añadir al final de `audience-ops/ideas/_inbox.md`. **Append, nunca rewrite**.

Si el fichero no existe (debería existir si la instancia fue creada por `init`), crearlo con la cabecera estándar.

### Paso 3 · Confirmar al usuario

Mostrar la línea añadida + ruta del fichero. Sin más.

## Pasos · Modo promote

### Paso 1 · Identificar origen

- Si el usuario pasó una línea del inbox: localizarla en `audience-ops/ideas/_inbox.md`.
- Si pasó un texto libre: tratarlo como ángulo de partida.

### Paso 2 · Interview de estructura

Preguntar (en este orden, con sugerencias cuando se pueda):

- **Slug** (kebab-case, único dentro de `audience-ops/ideas/`). Sugerir basado en el texto.
- **Pilar**: mostrar la lista de pilares de `audience-ops/strategy.md` y pedir uno. Si el usuario quiere uno fuera de la lista, advertir y permitir.
- **Canales destino**: mostrar la lista de canales activos (`audience-ops/channels/*.md`); pedir cero o más. Pueden añadirse después.
- **Hook / ángulo**: una o dos frases. Cómo se va a abordar la idea.
- **Notas / research**: opcional, contenido libre.

### Paso 3 · Crear `audience-ops/ideas/<slug>.md`

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

### Paso 4 · Marcar el origen en el inbox (si aplica)

Si la idea venía de una línea del inbox, **modificar esa línea** en `audience-ops/ideas/_inbox.md` añadiendo un sufijo de promoción:

Antes:
```
2026-05-19 · Comparar Garmin vs Whoop para zona 2
```

Después:
```
2026-05-19 · Comparar Garmin vs Whoop para zona 2  → ideas/garmin-vs-whoop-z2.md
```

No borrar la línea. Trazabilidad.

### Paso 5 · Confirmar al usuario

Mostrar:
- Ruta del fichero creado.
- Pilar asignado.
- Canales destino previstos.
- Recordatorio: "Cuando quieras draftear: `/audience-ops-draft <slug> <channel>`".

## Escritura · Ficheros creados o modificados

| Fichero | Modo | Acción |
|---|---|---|
| `audience-ops/ideas/_inbox.md` | quick | Append una línea |
| `audience-ops/ideas/_inbox.md` | promote | Modificar la línea origen (añadir sufijo) |
| `audience-ops/ideas/<idea-slug>.md` | promote | Crear |

## Criterios de éxito

### Modo quick
- `audience-ops/ideas/_inbox.md` tiene una línea nueva al final con prefijo `YYYY-MM-DD ·`.
- Ninguna otra línea fue modificada.

### Modo promote
- `audience-ops/ideas/<slug>.md` existe con frontmatter (`slug`, `pillar`, `channels`, `created`).
- Si vino de inbox, la línea origen lleva sufijo `→ ideas/<slug>.md`.
- Si el pilar no estaba en `strategy.md`, el usuario fue avisado y confirmó.

## Errores y casos límite

- **`audience-ops/` no existe**: abortar y sugerir `/audience-ops-init`.
- **Slug colisiona** con idea existente: pedir otro. Si el usuario insiste, ofrecer sufijo `-2`, `-3`.
- **`audience-ops/strategy.md` no existe** o no tiene pilares: permitir promote sin pilar (deja el campo vacío) y avisar de que se debería completar la estrategia.
- **No hay canales activos** para sugerir: permitir promote sin canales; pueden añadirse después editando el fichero.
- **Línea de inbox ya promovida** (ya tiene sufijo `→`): avisar y preguntar si se quiere crear una promoción adicional o abortar.
- **Texto de quick contiene saltos de línea**: convertirlos a un solo espacio. Una línea = una idea.

## Principios que aplica

- **Cero magia.** Las modificaciones del inbox (append, sufijo) se muestran como diff antes de aplicar si hay duda.
- **Single-instance por repo.** Una instancia bajo `audience-ops/`. No hay slug de proyecto que derivar — el "proyecto" es el repo host.
- **Append-only en el inbox** para captura rápida; en promote, edición controlada de una sola línea.
- **Frontmatter mínimo.** Sin campo `status` en la idea (se deriva del estado de sus publicaciones).

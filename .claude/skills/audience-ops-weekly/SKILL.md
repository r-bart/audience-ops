---
name: audience-ops-weekly
description: "Ritual semanal del operating system de contenidos: triage del inbox, vista de calendario, hygiene continua. Modo `--cleanup` para limpieza trimestral (archivado de publicadas viejas, drafts abandonados, ideas killed, proyectos paused)."
metadata:
  author: r-bart
  version: "0.2.0"
---

# weekly — Ritual semanal + cleanup trimestral

## Cuándo usar esta skill

- **Una vez por semana**: para cerrar el bucle (triage inbox → calendario → hygiene → decisiones de drafting).
- **Una vez por trimestre** (modo `--cleanup`): para mover lo viejo a `archive/`.
- Cuando notes que hay "ruido" acumulado (drafts viejos, ideas muertas en inbox, publicaciones programadas ya pasadas) y quieras barrerlo.

Es la única skill que **escribe estados** masivamente y mueve ficheros a `archive/`. Todo con confirmación.

## Entradas

- Opcional: `--cleanup` para modo trimestral. Sin esa flag, modo normal.

El "proyecto" es implícitamente la instancia local (`./audience-ops/`). Calendario, hygiene y cleanup operan sobre esta instancia. Cross-instance no soportado.

## Lectura previa

Antes de actuar, leer:

1. `audience-ops/config.yaml` — umbrales (`stale_draft_days`, `stale_inbox_days`, `cleanup_threshold_days`, `strategy_review_months`, `learnings_prompt_threshold_days`) y flags (`prompt_learnings_on_publish`).
2. Sobre la instancia local (`./audience-ops/`):
   - `audience-ops/ideas/_inbox.md` — entradas con prefijo de fecha.
   - `audience-ops/ideas/*.md` (excluir `archive/`) — ideas estructuradas, mirar `created` para staleness.
   - `audience-ops/publications/*.md` (excluir `archive/`) — frontmatter (`status`, `scheduled_for`).
   - `audience-ops/strategy.md` — `last_reviewed` (para sugerir revisión).

Si `audience-ops/` no existe, abortar y sugerir `/audience-ops-init`.

Las vistas activas **excluyen siempre cualquier ruta `*/archive/*`**.

## Detección de modo

- Sin flag → modo **normal** (ritual semanal).
- Con `--cleanup` → modo **cleanup** (trimestral).
- Si pasan ambos: ejecutar primero normal, luego cleanup.

## Convenciones sobre el inbox

Una línea = una idea (per SPEC §4). El inbox es **append-only y plano**: nunca se introducen headings ni se reordenan líneas. Las líneas se "anotan" con sufijos al final, manteniendo formato y orden originales:

- **Promovida**: `2026-05-19 · texto  → ideas/<slug>.md` (lo añade `audience-ops-idea --promote`).
- **Killed**: `2026-05-19 · texto  → killed 2026-05-28` (lo añade `weekly` modo normal al triage).
- **Pendiente** (sin sufijo): candidata a triage en el próximo `weekly`.

Tres sufijos posibles, mutuamente excluyentes. Una línea killed o promovida ya no vuelve a aparecer en triage.

## Pasos · Modo normal

### Paso 1 · Triage del inbox

Sobre la instancia local:

1. Leer `audience-ops/ideas/_inbox.md`.
2. Identificar entradas **sin sufijo** (las que no acaban en `→ ideas/...` ni `→ killed YYYY-MM-DD`).
3. Para cada entrada pendiente, mostrar al usuario:

```
[2026-05-21] Hook sobre HRV en zona 1
  → ¿promover, dejar, matar?
```

4. Acciones:
   - **promover** → invocar internamente la lógica de `audience-ops-idea --promote` (interview de slug/pilar/canales/hook). Marca la línea con sufijo `→ ideas/<slug>.md`.
   - **dejar** → no tocar. Volverá a salir en el próximo `weekly` con la misma fecha de captura original (no se renueva: la fecha es captura, no revisión).
   - **matar** → añadir sufijo `→ killed YYYY-MM-DD` (fecha de hoy) a la línea. Soft kill: la línea queda en `_inbox.md` permanentemente como histórico.

Si el usuario quiere triagear en bloque, ofrecer "saltar el resto de hoy" tras la primera entrada.

### Paso 2 · Vista de calendario

Recorrer `audience-ops/publications/*.md` (excluyendo `archive/`), parsear frontmatter, agrupar por **semana ISO** desde hoy.

Renderizar tabla:

```
Semana 22 (1-7 jun)
  · lun  newsletter   verxion       cardio-rmssd       (ready)
  · mié  x            verxion       garmin-vs-whoop-z2 (draft)
Semana 23 (8-14 jun)
  · jue  linkedin     audience-ops  why-i-killed-X     (ready)
```

Reglas:
- Items sin `scheduled_for` → no aparecen.
- Items con `status: published` → no aparecen (ya hechos).
- Items con `scheduled_for` en el pasado y `status: ready` → marcar `(overdue)` y resaltar (van a la sección hygiene).
- Vista de la instancia local. Sin argumentos de filtrado de proyecto (no aplica en single-instance).

Detección de **conflicto de slot**: si dos publicaciones de mismo canal coinciden en la misma fecha, avisar.

Detección de **dispersión per-canal**: por canal de la instancia, contar items en las próximas 4 semanas; comparar con `cadence` de `audience-ops/channels/<id>.md`. Si por debajo (ej. newsletter `1/week` con 0 ready en 4 semanas), avisar.

### Paso 3 · Hygiene continua

Tres tipos de aviso, agrupados:

#### 3a · Drafts viejos

Buscar publicaciones con `status: draft` y `mtime` > `stale_draft_days` (default 30).

Para cada uno, mostrar y preguntar: ¿iterar / abandonar / dejar?

- **iterar** → sugerir al usuario invocar `/audience-ops-draft <idea> <channel>` para regenerar.
- **abandonar** → mover el fichero **inmediatamente** a `audience-ops/publications/archive/abandoned/<idea-slug>-<channel>.md`, conservando frontmatter y contenido (no se cambia `status`, el path comunica el estado). Acción confirmada antes de mover.
- **dejar** → no tocar. Volverá a salir el próximo `weekly` (mtime sigue siendo el mismo).

#### 3b · Ready vencidos

Buscar publicaciones con `status: ready` y `scheduled_for < hoy`.

Para cada uno, mostrar y preguntar:
- ¿Lo publicaste? → cambiar a `status: published` (preserva el `scheduled_for` como fecha efectiva).
- ¿Lo cancelaste? → cambiar a `status: draft` y limpiar `scheduled_for` (vuelve a la cola de drafts). Si es definitivo, ofrecer abandonar (mover a `archive/abandoned/`, mismo flujo que 3a).
- ¿Lo reprogramaste? → pedir nueva fecha, actualizar `scheduled_for`.

**Después de confirmar el cambio a `published` (y solo si `prompt_learnings_on_publish: true` en config.yaml, default true)**:

> "¿algo que aprendiste de esta? (puedes saltar)"

Si el usuario aporta bullets, **append** un `## Aprendizajes` al final del cuerpo del fichero de la publicación con esos bullets (texto literal de cada bullet, sin interpretar markdown como headings). Si la sección ya existía (caso raro: aprendizajes añadidos manualmente antes del weekly), agregar bullets al final de la sección existente. Si el usuario salta, no se crea sección.

Si `prompt_learnings_on_publish: false`, el prompt se suprime — el usuario puede añadir aprendizajes a mano editando el fichero o esperar a `weekly --cleanup` (Cleanup 5) para barrido retroactivo.

#### 3c · Inbox stale

Buscar entradas pendientes (sin sufijo) en `_inbox.md` con prefijo de fecha > `stale_inbox_days` (default 30).

Para cada una, preguntar: ¿promover / matar / dejar?

- **promover** y **matar**: mismas acciones que en Paso 1 (sufijo correspondiente).
- **dejar** → no tocar. La fecha de captura se mantiene (no se renueva). Volverá a salir como stale el próximo weekly hasta que el usuario decida.

### Paso 4 · Decisiones de drafting

Listar ideas estructuradas (`audience-ops/ideas/<slug>.md`, excluyendo `archive/`) que **no tienen aún** una publicación correspondiente en alguno de sus `channels:` del frontmatter.

Para cada combinación idea + canal pendiente, ofrecer:
- Draftear ahora → sugerir `/audience-ops-draft <idea-slug> <channel>`.
- Dejar para otra semana → no hacer nada.

Esto cierra el bucle: las ideas que llevan tiempo estructuradas sin draftearse aparecen aquí cada semana.

### Paso 5 · Resumen del ritual

Mostrar al usuario:
- Entradas del inbox triageadas (promovidas / dejadas / killed).
- Cambios de estado aplicados (ready → published, ready → draft con reset de `scheduled_for`, reagendados).
- Drafts movidos a `archive/abandoned/`.
- Decisiones de drafting tomadas para esta semana.
- Avisos pendientes que el usuario marcó "dejar" (para que vuelva a ver en el próximo `weekly`).

## Pasos · Modo `--cleanup` (trimestral)

Cinco acciones, **todas con confirmación**, en este orden:

### Cleanup 1 · Publicadas viejas

Buscar `status: published` con `scheduled_for` anterior a `hoy - cleanup_threshold_days` (default 90).

Para cada bloque por año, mostrar lista resumida y preguntar: "¿archivar las N publicaciones de YYYY?". En afirmativa, mover a `audience-ops/publications/archive/<año>/<idea-slug>-<channel>.md`. Mantener nombre de fichero original.

### Cleanup 2 · Drafts muy viejos no revisados

Drafts en estado `status: draft` con `mtime` > 2 × `stale_draft_days` (default 60). Son drafts que el usuario llevaba ignorando incluso en weeklies (no los abandonó, pero tampoco iteró).

Mostrar lista y para cada uno preguntar: ¿iterar / abandonar (mover a `archive/abandoned/`) / dejar?

Misma mecánica que hygiene 3a, sólo que el umbral es más agresivo y se ejecuta como barrido trimestral.

### Cleanup 3 · Ideas estructuradas killed

Buscar ideas estructuradas (`audience-ops/ideas/<slug>.md`) que ya no tienen ninguna publicación viva (todas sus publicaciones referenciadas en `channels:` del frontmatter están en `archive/abandoned/` o nunca existieron y la idea lleva > `stale_inbox_days` sin actividad).

Preguntar al usuario por cada una: ¿archivar? En afirmativa, mover `audience-ops/ideas/<slug>.md` a `audience-ops/ideas/archive/`.

**El `_inbox.md` no se toca en cleanup**: las líneas killed (con sufijo `→ killed YYYY-MM-DD`) viven indefinidamente en `_inbox.md` como histórico permanente, sin coste — son una línea cada una y aportan trazabilidad de qué descartaste y cuándo.

### Cleanup 4 · Revisión de estrategia

Para cada `audience-ops/strategy.md` con `last_reviewed` > `strategy_review_months` (default 4) en el pasado:

Avisar: "La estrategia lleva X meses sin revisar. ¿Lanzar `/audience-ops-strategy` ahora?".

No archivar nada — la estrategia no se "cumple", se refresca.

### Cleanup 5 · Publicaciones sin aprendizajes

Buscar publicaciones con `status: published` cuyo `scheduled_for` es anterior a `hoy - learnings_prompt_threshold_days` (default 60) y que **no tienen** una sección `## Aprendizajes`.

Listar al usuario. Para cada una, preguntar: "¿añadir aprendizajes ahora? (skip / dictar bullets / pasar al siguiente)".

- **Dictar bullets** → append `## Aprendizajes` al cuerpo del fichero con los bullets dictados (texto literal, sin interpretar markdown como headings).
- **Skip** → no se toca. Volverá a salir en el próximo `--cleanup` (su edad sigue creciendo).
- **Pasar al siguiente** → marcar internamente como "ya preguntada esta vez" y omitir resto del lote.

Este paso es **barrido retroactivo** para publicaciones donde no se capturó aprendizajes en su momento (o porque `prompt_learnings_on_publish` estaba en false, o porque el ritual semanal no se corrió cuando tocaba).

### Resumen final del cleanup

Mostrar conteos: N publicaciones archivadas, M drafts abandonados, K ideas killed, L revisiones de estrategia sugeridas, P publicaciones con aprendizajes retroactivos.

## Escritura · Ficheros creados o modificados

| Fichero | Modo | Acción |
|---|---|---|
| `audience-ops/ideas/_inbox.md` | normal | Sufijar líneas: `→ ideas/<slug>.md` (promovidas) o `→ killed YYYY-MM-DD` (killed) |
| `audience-ops/publications/<...>.md` | normal | Cambios de `status` y `scheduled_for` en frontmatter |
| `audience-ops/publications/archive/abandoned/<...>.md` | normal o cleanup | Mover desde `publications/` (al abandonar drafts) |
| `audience-ops/publications/archive/<año>/<...>.md` | cleanup | Mover published viejas |
| `audience-ops/ideas/archive/<slug>.md` | cleanup | Mover ideas estructuradas killed |
| `audience-ops/publications/<...>.md` | normal o cleanup | Append `## Aprendizajes` section con bullets cuando el usuario los dicta |

## Criterios de éxito

### Modo normal
- Toda entrada del inbox sin sufijo se mostró al usuario para decisión.
- Las publicaciones con `scheduled_for` pasado en estado `ready` se reconciliaron (published, draft, reagendado, o explícitamente dejado).
- Los drafts > `stale_draft_days` se mostraron al usuario; los abandonados se movieron a `archive/abandoned/` con confirmación.
- Se rindió un calendario coherente con los `scheduled_for` actuales.

### Modo cleanup
- Toda publicación con `scheduled_for > cleanup_threshold_days` en pasado y `status: published` se evaluó (archivada o dejada por decisión explícita).
- Los drafts ultra-viejos (> 2 × `stale_draft_days`) se evaluaron.
- Las ideas estructuradas sin publicaciones vivas se evaluaron para archivado.
- Las strategy stale se reportaron.
- Las publicaciones published > `learnings_prompt_threshold_days` sin `## Aprendizajes` se evaluaron para barrido retroactivo.

## Errores y casos límite

- **`audience-ops/` no existe**: avisar, sugerir `/audience-ops-init`.
- **Una publicación tiene frontmatter inválido** (yaml roto, `status` desconocido): mostrar al usuario el fichero, no intentar reparar; tratarla como "skipped" en este ritual.
- **`audience-ops/config.yaml` sin umbrales o flags**: usar los defaults documentados aquí (`stale_draft_days: 30`, `stale_inbox_days: 30`, `cleanup_threshold_days: 90`, `strategy_review_months: 4`, `learnings_prompt_threshold_days: 60`, `prompt_learnings_on_publish: true`) y avisar al usuario una vez ("usando defaults; ajústalos en `audience-ops/config.yaml` si quieres").
- **Múltiples secciones `## Aprendizajes` en un mismo fichero de publicación** (caso raro por edit manual): consolidar bajo la **primera** sección y avisar al usuario ("publication X tenía 2+ secciones `## Aprendizajes`; bullets nuevos van a la primera, recomiendo consolidar a mano"). No reordenar ni borrar las secciones duplicadas — el usuario decide cómo consolidar.
- **Conflicto de slot** (dos publicaciones mismo canal misma fecha): avisar, no resolver automáticamente — el usuario decide cuál mueve.
- **El usuario corta el ritual a mitad**: los cambios ya confirmados quedan aplicados. Lo no triageado vuelve a aparecer la próxima vez. Cero estado intermedio.
- **`archive/` ya existe con estructura distinta** (ej. fichero llamado `archive` en lugar de carpeta): avisar, no sobreescribir.
- **Inbox sin prefijo de fecha** en alguna línea: no asumir staleness; mostrar al usuario para que aclare.
- **Línea del inbox con sufijo desconocido** (ej. usuario lo editó a mano): no asumir; mostrar al usuario.

## Principios que aplica

- **Cero magia.** Nada se mueve, archiva ni cambia de status sin confirmación explícita.
- **Soft archive, never delete.** Todo lo que se "quita" se mueve a un directorio `archive/` en la jerarquía adecuada. Reversible con un `git mv`.
- **Vistas regenerables.** El calendario, los conteos de cadencia y los avisos de hygiene se computan al vuelo desde los ficheros — no hay caché ni índice.
- **Una sola fuente de verdad por cosa.** El estado vive en frontmatter de publicaciones; el "abandono" vive en path (`archive/abandoned/`), no en un cuarto estado fantasma. Las decisiones del inbox viven en sufijos de línea, no en headings ni en ficheros separados.
- **Append-only en el inbox.** Las líneas nunca se eliminan, sólo se anotan con sufijos. El histórico de qué killed y cuándo es trazable forever.
- **El proyecto se deriva del path.** Las vistas globales agregan, no duplican slug en frontmatter de publicaciones.
- **Convención sobre configuración.** Los umbrales tienen defaults razonables; `config.yaml` solo cambia lo que el usuario quiera distinto.

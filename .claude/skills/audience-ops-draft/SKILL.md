---
name: audience-ops-draft
description: "Convierte una idea estructurada en un draft listo para un canal concreto. Aplica voz, formato del canal y ángulo de la idea. Soporta repurpose de una publicación existente a otro canal."
metadata:
  author: r-bart
  version: "0.3.0"
---

# draft — Idea + canal → publicación

## Cuándo usar esta skill

- Tienes una idea estructurada (en `audience-ops/ideas/<idea>.md`) y quieres convertirla en un draft publicable para un canal concreto.
- Tienes una publicación existente y quieres **repurposear** su contenido a otro canal (la misma skill, modo `--from`).

## Entradas

- **idea-slug**: el slug del fichero de idea (sin `.md`).
- **channel**: id del canal destino (debe existir en `audience-ops/channels/<channel>.md`).
- Opcional **--from `<publication-path>`**: ruta a una publicación ya existente que se quiere adaptar. Si se pasa, la idea-slug puede inferirse del frontmatter `idea:` de esa publicación.
- Opcional **--to `<ch1,ch2,...>`** (requiere `--from`): lista comma-separated de canales destino para **batch repurpose**. Transforma la publicación origen de `--from` a cada uno de los canales destino en una sola invocación, generando un fichero por canal. Pre-flight: si algún canal listado no existe en el proyecto, abortar antes de generar nada. **`--to` sin `--from` es error**: el flujo single-channel desde una idea pelada se invoca como `draft <idea> <channel>` sin flags.
El "proyecto" es implícitamente la instancia local (`./audience-ops/`). No hay slug ni selección.

## Lectura previa

### Paso previo · Dossier de proyecto (v0.3.0+, si aplica)

Antes del resto de lecturas:

1. Listar `audience-ops/projects/*.md` (excluir `archive/`). Si el directorio no existe, omitir todo este paso previo.
2. Para cada dossier encontrado, parsear su frontmatter y construir un mapa `slug_prefix: <dossier-path>`.
3. Extraer el prefix del `idea-slug` partiendo por el primer `-`. Si la idea es `bl-cloudflare-migration`, el prefix candidato es `bl`.
4. Si el prefix matchea exactamente un dossier en el mapa, marcar ese dossier para inclusión.
5. Si el prefix matchea **2+ dossiers** (colisión rara por edit manual): warning al usuario ("Found 2 dossiers with prefix `bl`: …"), usar el primer match alfabéticamente.
6. Si el prefix no matchea ningún dossier (incluyendo el caso de idea-slug sin `-`): seguir adelante sin dossier, sin warning.

Si un dossier matcheado tiene frontmatter inválido (YAML roto): warning y omitir ese dossier; no bloquear el resto del draft.

### Lecturas estándar

Antes de generar nada, leer:

1. `audience-ops/voice.md` — voz y tono del proyecto.
2. `audience-ops/channels/<channel>.md` — frontmatter (handle, cadencia) + formato + ajustes de voz por canal.
3. `audience-ops/ideas/<idea-slug>.md` — frontmatter + hook + notas.
4. Si modo `--from`: la publicación origen.
5. `audience-ops/strategy.md` — para conocer pilares y aplicar coherencia (no para generar el draft directamente).

Si **alguna** de las tres primeras lecturas falla, abortar con mensaje claro al usuario (no improvisar).

## Pasos

### Paso 1 · Validación

- Confirmar que el canal existe en `channels/`.
- Confirmar que la idea existe en `ideas/`.
- Si el frontmatter de la idea lista `channels:` y el canal solicitado no está → avisar al usuario (no es un error, pero merece la pena confirmar).
- Si ya existe `audience-ops/publications/<idea>-<channel>.md`:
  - Mostrar al usuario lo que ya hay.
  - Preguntar: ¿sobreescribir, abandonar, o tratar esta invocación como **iteración** (modificar lo existente)?

### Paso 2 · Componer contexto

Construir el contexto de generación con:

- **Project dossier** (si fue matcheado en el paso previo): contenido completo de `audience-ops/projects/<project-slug>.md`. Úsalo para referencias factuales del producto (stack, audiencia, story log, ángulos abiertos). **El ángulo de la idea y el formato del canal mantienen prioridad** cuando hay conflicto — el dossier informa, no dicta.
- **Voz base**: contenido de `voice.md`.
- **Ajustes de voz del canal**: sección `## Ajustes de voz` de `channels/<channel>.md`, si existe. Estos overridean la voz base.
- **Formato del canal**: sección `## Formato` de `channels/<channel>.md`.
- **Sí / No del canal**: secciones si existen.
- **Hook y notas de la idea**: secciones `## Hook / ángulo` y `## Notas / research`.
- **Pilar**: para coherencia tonal con el pilar al que pertenece la idea.
- Modo `--from`: el contenido de la publicación origen como base a transformar.

### Paso 3 · Generar el draft

Producir el contenido respetando:

- **Formato del canal** estrictamente (longitud, número de partes, estructura).
- **Voz** (base + overrides).
- **Sí / No** del canal — no hacer lo que está en "No".
- **Ángulo** de la idea — no inventar un ángulo distinto.

#### Particularidades por canal

- **Newsletter**: además del cuerpo, generar `subject` (≤50 chars, sin clickbait) y `preheader` (complementa, no repite). Ambos van en el frontmatter de la publicación.
- **X / Twitter**: si es thread, separar los tweets con líneas en blanco en el cuerpo del fichero. **No** numerar con (1/n), (2/n) salvo que el formato del canal lo pida.
- **LinkedIn**: párrafos cortos, primera línea es hook, sin emojis salvo que el formato del canal los autorice.
- **Blog**: estructura completa con H1 (título) + cuerpo. El título también va en frontmatter como `title`.

Si el canal `id` no está en esta lista, regirse exclusivamente por lo que diga `channels/<channel>.md`.

### Paso 4 · Construir frontmatter de la publicación

Base común:

```yaml
---
idea: <idea-slug>
channel: <channel>
status: draft
scheduled_for: <vacío inicialmente, lo asigna weekly o el usuario>
---
```

Añadir campos específicos del canal:

- newsletter: `subject`, `preheader`
- blog: `title`

### Paso 5 · Escribir el fichero

Ruta: `audience-ops/publications/<idea-slug>-<channel>.md`.

Antes de escribir, mostrar al usuario:
- Path completo.
- El contenido propuesto (frontmatter + cuerpo).

Pedir confirmación.

### Paso 6 · Revisión guiada (Ready check)

Tras escribir el draft, **proponer una revisión** con esta checklist (puede ser un mensaje al usuario o auto-evaluación si el agente puede):

- [ ] **Hook**: ¿el primer tweet/párrafo/línea engancha con un claim concreto, no una pregunta retórica?
- [ ] **Longitud**: ¿respeta el rango del canal?
- [ ] **Voz**: ¿suena al `voice.md` del proyecto, no a marketing genérico?
- [ ] **Pilar**: ¿queda claro a qué pilar pertenece?
- [ ] **Sí / No del canal**: ¿no toca ninguna prohibición del canal?
- [ ] **Newsletter solo**: ¿`subject` y `preheader` complementan en vez de repetir?
- [ ] **Cero placeholders**: ¿no hay `[texto aquí]`, `TODO`, etc.?

Si **todas** pasan, ofrecer cambiar el `status` de `draft` a `ready`. Si alguna falla, dejar como `draft` y listar qué arreglar.

### Paso 7 · Resumen final

- Path del fichero.
- Estado final (`draft` o `ready`).
- Recordatorio del próximo paso:
  - Si `draft`: "Itera sobre el fichero o reinvoca `/audience-ops-draft` para regenerar".
  - Si `ready`: "Cópialo en <scheduler de `config.yaml`> cuando vayas a programarlo; `weekly` te lo recordará".

## Modo repurpose (`--from`)

Cuando se pasa `--from <publication-path>`:

1. Leer la publicación origen entera (frontmatter + cuerpo).
2. Extraer `idea:` de su frontmatter; usarlo si no se pasó `idea-slug` explícitamente.
3. En lugar de generar desde la idea pelada, **transformar** el cuerpo de la publicación origen al formato del canal destino.
4. Mantener el hook/ángulo de la origen pero adaptar la estructura, longitud y voz a las reglas del canal destino.
5. El resto del flujo (frontmatter, revisión, status) es idéntico al modo normal.

**Importante**: el repurpose no es copy-paste. Si el contenido no se puede adaptar al canal destino sin perder el ángulo, decírselo al usuario y abortar.

## Modo batch (`--to`)

Cuando se pasa `--to <ch1,ch2,...>` **junto con `--from <publication-path>`**: batch repurpose de una publicación origen a N canales destino en una sola invocación.

`--to` sin `--from` no está soportado (ver "Errores y casos límite"). Para draftear una idea desde cero a varios canales, se invoca `draft` una vez por canal.

### Pre-flight

1. Parsear la lista comma-separated en N canal-ids.
2. Para cada `<channel>`, verificar que existe `audience-ops/channels/<channel>.md`. Si **alguno** falta, abortar listando los que faltan. Sin esto se genera basura.
3. Validar que ningún canal destino aparece duplicado (warning, no error).
4. Leer la publicación origen una vez (vía `--from`).

### Generación secuencial

Para cada canal destino, en orden de aparición:

1. Ejecutar el flujo de generación de modo normal (lectura de voice + canal + idea + opcional origen → componer contexto → generar → mostrar → confirmar → escribir → review).
2. Cada draft escribe `audience-ops/publications/<idea-slug>-<channel>.md`.
3. Si el fichero ya existe, aplica el flujo existente "sobreescribir / iterar / abandonar" para ese canal individual. No se aplica blanket-yes para el batch.
4. Si la review checklist falla en un canal individual, ese fichero queda como `draft` con notas; el siguiente canal sigue su curso sin verse afectado.

### Particularidades

- **Rechazo individual del usuario** (decir "no" al confirmar un draft generado): el fichero NO se escribe, el siguiente canal procede.
- **Abort mid-batch** (usuario interrumpe): los drafts ya confirmados/escritos quedan; el resumen final refleja qué se completó.
- **Canal destino == canal del origen** (en modo `--from`): warning ("estás regenerando el canal origen — ¿continúas?"), permitir tras confirmación. Útil cuando se quiere reescribir el origen con un ángulo distinto.

### Resumen final

Tras el batch, renderizar tabla:

| Canal destino | Resultado | Path |
|---|---|---|
| linkedin | ready | `audience-ops/publications/<idea>-linkedin.md` |
| x | draft (review failed: subject) | `audience-ops/publications/<idea>-x.md` |
| blog | rechazado por el usuario | (no escrito) |

## Escritura · Ficheros creados o modificados

| Fichero | Acción |
|---|---|
| `audience-ops/publications/<idea-slug>-<channel>.md` | Crear (o sobreescribir/iterar tras confirmación) |

## Criterios de éxito

- El fichero existe con frontmatter completo y válido.
- El cuerpo respeta el formato del canal (longitud y estructura).
- La revisión guiada ha sido presentada al usuario.
- El `status` refleja honestamente la calidad del draft (no marcar `ready` si la revisión no pasó).

## Errores y casos límite

- **Idea no existe**: abortar con mensaje. Sugerir crearla con `/audience-ops-idea`.
- **Canal no existe en el proyecto**: abortar. Listar canales disponibles.
- **`voice.md` vacío o `_pendiente_`**: avisar de que el draft saldrá sin restricciones de voz; pedir confirmación al usuario antes de continuar.
- **Idea sin hook**: pedir al usuario que añada un ángulo a la idea antes de draftear; o, si insiste, generar con un hook tentativo y marcarlo en la revisión guiada.
- **Newsletter sin generar `subject` válido** (>50 chars): no marcar `ready`. Pedir al usuario que afine o regenerar.
- **Repurpose desde un canal muy distinto** (ej. newsletter → X): si la longitud no entra, ofrecer dividir en varias publicaciones (no asumir).
- **El draft generado contiene "anti-temas"** listados en `strategy.md`: detectar y avisar antes de escribir.
- **Prefix de idea-slug no matchea ningún dossier**: proceder sin dossier, sin warning. La convención de prefix es opcional.
- **Prefix colisiona con 2+ dossiers**: warning + usar el primero alfabéticamente.
- **Dossier con frontmatter YAML inválido**: warning + omitir ese dossier de la composición de contexto; no bloquear el draft.
- **`audience-ops/projects/` no existe**: comportamiento idéntico a v0.2.0 (no dossiers, no warnings, paso previo se omite entero).
- **Modo `--to` sin `--from`**: error con mensaje "`--to` requiere `--from <publication-path>`. Para draftear una idea a varios canales desde cero, invoca `draft <idea> <channel>` una vez por canal." No generar nada.
- **Modo `--to` con canal inexistente**: pre-flight aborta listando todos los canales faltantes. No se genera nada.
- **Modo `--to` con un solo canal**: behavior idéntico a modo single-channel `--from`; el flag se acepta por simetría.
- **Modo `--to` cuyo lista incluye el canal del origen `--from`**: warning de regeneración del origen, permitir con confirmación explícita.
- **Modo `--to` con interrupt mid-batch**: los drafts confirmados quedan escritos; el resto se omite; resumen lista lo completado.

## Principios que aplica

- **Cero magia.** El draft se muestra antes de escribir. La transición a `ready` requiere que la revisión pase.
- **Una sola fuente de verdad por cosa.** El estado vive en frontmatter. El proyecto se deriva del path.
- **Voz primero, formato segundo, ángulo siempre.** Las tres entradas (`voice.md`, `channels/<id>.md`, idea) ordenadas por prioridad cuando entran en conflicto: el ángulo de la idea manda; las reglas del canal definen forma; la voz colorea el todo.
- **Sin placeholders en el output.** Un draft con `[completar aquí]` no es un draft, es una factura para el futuro.

---
name: audience-ops-draft
description: "Convierte una idea estructurada en un draft listo para un canal concreto. Aplica voz, formato del canal y ángulo de la idea. Soporta repurpose de una publicación existente a otro canal."
metadata:
  author: r-bart
  version: "0.10.1"
---

# draft — Idea + canal → publicación

## Cuándo usar esta skill

- Tienes una idea estructurada (en `projects/<slug>/ideas/<idea>.md`) y quieres convertirla en un draft publicable para un canal concreto.
- Tienes una publicación existente y quieres **repurposear** su contenido a otro canal (la misma skill, modo `--from`).

## Entradas

- **idea-slug**: el slug del fichero de idea (sin `.md`).
- **channel**: id del canal destino (debe existir en `projects/<slug>/channels/<channel>.md`).
- Opcional **--from `<publication-path>`**: ruta a una publicación ya existente que se quiere adaptar. Si se pasa, la idea-slug puede inferirse del frontmatter `idea:` de esa publicación.
- Opcional **slug del proyecto**. Si no, usar `defaults.project` de `config.yaml`. Si no, preguntar.

## Lectura previa

Antes de generar nada, leer:

1. `projects/<slug>/voice.md` — voz y tono del proyecto.
2. `projects/<slug>/channels/<channel>.md` — frontmatter (handle, cadencia) + formato + ajustes de voz por canal.
3. `projects/<slug>/ideas/<idea-slug>.md` — frontmatter + hook + notas.
4. Si modo `--from`: la publicación origen.
5. `projects/<slug>/strategy.md` — para conocer pilares y aplicar coherencia (no para generar el draft directamente).

Si **alguna** de las tres primeras lecturas falla, abortar con mensaje claro al usuario (no improvisar).

## Pasos

### Paso 1 · Validación

- Confirmar que el canal existe en `channels/`.
- Confirmar que la idea existe en `ideas/`.
- Si el frontmatter de la idea lista `channels:` y el canal solicitado no está → avisar al usuario (no es un error, pero merece la pena confirmar).
- Si ya existe `projects/<slug>/publications/<idea>-<channel>.md`:
  - Mostrar al usuario lo que ya hay.
  - Preguntar: ¿sobreescribir, abandonar, o tratar esta invocación como **iteración** (modificar lo existente)?

### Paso 2 · Componer contexto

Construir el contexto de generación con:

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

Ruta: `projects/<slug>/publications/<idea-slug>-<channel>.md`.

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

## Escritura · Ficheros creados o modificados

| Fichero | Acción |
|---|---|
| `projects/<slug>/publications/<idea-slug>-<channel>.md` | Crear (o sobreescribir/iterar tras confirmación) |

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

## Principios que aplica

- **Cero magia.** El draft se muestra antes de escribir. La transición a `ready` requiere que la revisión pase.
- **Una sola fuente de verdad por cosa.** El estado vive en frontmatter. El proyecto se deriva del path.
- **Voz primero, formato segundo, ángulo siempre.** Las tres entradas (`voice.md`, `channels/<id>.md`, idea) ordenadas por prioridad cuando entran en conflicto: el ángulo de la idea manda; las reglas del canal definen forma; la voz colorea el todo.
- **Sin placeholders en el output.** Un draft con `[completar aquí]` no es un draft, es una factura para el futuro.

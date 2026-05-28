---
name: audience-ops-strategy
description: "Interview para crear o actualizar el `strategy.md` de un proyecto (posicionamiento, ICP, pilares, objetivos, anti-temas). Avisa si `last_reviewed` lleva más del umbral configurado."
metadata:
  author: r-bart
  version: "0.11.1"
---

# strategy — Crear o actualizar la estrategia de un proyecto

## Cuándo usar esta skill

- El proyecto no tiene `strategy.md` aún (o solo tiene el placeholder que dejó `init` en modo bootstrap temprano).
- Quieres revisar / actualizar una estrategia existente: el contexto cambió, los pilares ya no encajan, los objetivos del trimestre son otros.
- `weekly --cleanup` te avisó de que `last_reviewed` lleva más de `strategy_review_months` (default 4) sin tocarse.

Esta skill **no toca** `voice.md` ni los canales — son ficheros separados (ver "Conceptos clave" abajo).

## Conceptos clave

Antes del interview, contexto sobre los términos que aparecen — todos son fuente común de fricción para quien empieza.

### ICP (*Ideal Customer Profile*)

El lector exacto al que escribes. No vale "developers" ni "gente que entrena"; vale algo concreto como "senior backend engineer en startup de 10-50 personas que escribe Go a diario y nunca ha tocado React" o "atleta amateur 30-40 años con wearable, frustrada porque sus métricas no se traducen en mejor planificación".

Es jerga prestada de B2B sales. Si te incomoda el término, mentalmente sustitúyelo por "lector ideal". El skill fuerza esta precisión porque la ambigüedad aquí se propaga: `idea --promote` necesita decidir si una idea encaja con el ICP, `draft` necesita escribir con un lector concreto en la cabeza, y "gente que entrena" no es lo bastante específico para dirigir un ángulo.

### Objetivos ≠ OKRs

Son **intenciones con número**, no la metodología OKR formal. No hace falta jerarquía objetivo/key-result, ni 3-5 de cada, ni revisión cuantitativa rigurosa. Formato sugerido y suficiente:

```
<periodo>: <métrica concreta>
```

Ejemplos:
- "Q3 2026: 12 newsletters publicadas, 200 suscriptores nuevos."
- "H1 2027: 5 piezas long-form, posicionar para 'HRV training'."

Sirven para dos cosas: (a) que `weekly` contraste tu actividad real con la cadencia que dijiste querer, (b) que cuando revises la estrategia a los 4 meses, decidas si los pilares siguen aportando hacia esos objetivos. Si no quieres meterte en esto, déjalo en `_pendiente_` (ver siguiente punto).

### Estructural vs contenido — qué es opcional

Las 5 secciones (Posicionamiento, Audiencia/ICP, Pilares, Objetivos, Anti-temas) son **estructuralmente obligatorias**: todas existen en `strategy.md` siempre, aunque sea con un placeholder. El **contenido de cada sección es opcional**: puedes dejar `_pendiente_` y refinar después en otra invocación de `strategy`.

Excepción práctica: **Pilares** sin al menos uno deja a `idea --promote` y `draft` operando a ciegas (sin lista contra la que validar el `pillar:` de cada idea). El skill te deja seguir sin pilares pero te avisa.

Por qué la estructura es fija (principio "Convención sobre configuración" del SPEC): cualquier skill o agente puede ir a buscar "## Objetivos" en cualquier `strategy.md` y saber que estará — aunque esté vacía. Cero adivinanza estructural.

**Una nota sobre aprendizajes**: en modo update, strategy lee las secciones `## Aprendizajes` de las publicaciones de cada pilar y te ofrece refinar la descripción del pilar basándote en lo que aprendiste en uso real. Sin aprendizajes, el flujo sigue normal — solo es un acelerador, no un requisito.

### Por qué no se toca `voice.md` ni canales

Estrategia y voz son **ortogonales**:

| | Estrategia | Voz |
|---|---|---|
| Pregunta que responde | **Qué** cuentas y **a quién** | **Cómo** lo cuentas |
| Cambia cuando | El ICP cambia o sumas pilar | Casi nunca; tu voz es tuya |
| Entre proyectos | Estrategia única por proyecto | Misma voz puede aplicar a varios |

Mezclarlas en un fichero invalidaría `last_reviewed` cada vez que tocas una coma de la voz (y al revés). Canales son otra dimensión más (formato + cadencia + ajustes de voz por plataforma) y viven en `channels/<id>.md`. Una skill = un concepto. La voz se edita a mano hoy; `voice.md` es corto.

## Entradas

- Opcional: slug del proyecto. Si no se da, usar `defaults.project` de `config.yaml`. Si tampoco, preguntar mostrando los activos de `portfolio.yaml`.
- Opcional: bloque concreto a revisar (`positioning`, `icp`, `pillars`, `goals`, `anti-topics`). Si no se da, recorrer todos.

## Lectura previa

Antes de actuar, leer:

1. `config.yaml` — para `defaults.project` y `strategy_review_months`.
2. `portfolio.yaml` — para validar que el proyecto existe y leer su `one_liner` (contexto inicial del interview).
3. `projects/<slug>/strategy.md` — si existe, para modo update.
4. `projects/<slug>/publications/*.md` (excluir `archive/`) — leer las secciones `## Aprendizajes` agrupadas por pilar (vía `idea:` del frontmatter de publicación → `pillar:` del frontmatter de la idea) para el modo update de bloque 3.3.

Si **no existe** `projects/<slug>/strategy.md` y tampoco el directorio del proyecto: abortar y sugerir `/audience-ops-init`.

## Detección de modo

- `strategy.md` no existe (o solo contiene `_pendiente_` en todas las secciones) → **modo create**: interview completo, todas las secciones obligatorias.
- `strategy.md` existe con secciones rellenas → **modo update**: para cada bloque, mostrar lo actual y preguntar "¿mantener / refinar / reescribir / saltar?".

## Pasos

### Paso 1 · Determinar proyecto

- Si el usuario indicó proyecto → usar.
- Si no, `defaults.project` de `config.yaml`.
- Si no hay default → preguntar listando proyectos activos de `portfolio.yaml`.

### Paso 2 · Comprobar staleness (solo modo update)

Si existe `strategy.md` con frontmatter `last_reviewed`:

- Calcular meses desde `last_reviewed` hasta hoy.
- Si pasa de `strategy_review_months` (default 4): avisar al usuario antes de empezar ("Última revisión hace X meses, igual conviene mirarla entera").
- Si no, ofrecer modo bloque-específico ("¿Revisar todo, o solo `pillars`/`goals`/`anti-topics`?").

### Paso 3 · Interview por bloques

Para cada bloque, en este orden:

#### 3.1 · Posicionamiento

Pregunta guía: "En una frase, ¿qué es este proyecto y para quién?"

Refinamiento:
- ¿Qué problema resuelve?
- ¿Qué hace distinto de las alternativas que el usuario consideraría?
- ¿Qué cambio promete en la vida del usuario?

Si en modo update, mostrar lo que hay y pedir: mantener / refinar / reescribir / saltar.

#### 3.2 · Audiencia / ICP

> Ver "Conceptos clave · ICP" si el término no te dice nada. Bottom line: lector ideal con detalle concreto.

Pregunta guía: "¿Quién es el lector ideal? Descríbelo concreto."

Refinamiento:
- Rol / contexto profesional.
- Qué le pasa el día que decide actuar (el "trigger" de uso).
- Qué ya ha probado y no le funcionó.
- Dónde lee actualmente (canales que ya consume).

#### 3.3 · Pilares

Pregunta guía: "¿Sobre qué 3 temas vas a escribir sistemáticamente?"

Para cada pilar (entre 2 y 5; si pide más de 5, advertir que se diluye):
- **Slug del pilar** (kebab-case, ej. `behind-the-build`).
- **Descripción** (una frase: ¿qué se cuenta bajo este pilar y qué no?).
- **Por qué este pilar y no otro** (criterio de inclusión).

Si en modo update, mostrar pilares actuales y permitir añadir / eliminar / renombrar / reescribir descripción.

**En modo update, antes de iterar cada pilar**:

1. Recorrer `projects/<slug>/publications/*.md` (excluyendo `archive/`).
2. Para cada publicación, leer su `idea:` del frontmatter y buscar el fichero de idea correspondiente en `projects/<slug>/ideas/<idea-slug>.md`. Si la idea existe y su frontmatter tiene `pillar: <este-pilar>`, extraer los bullets de la sección `## Aprendizajes` de la publicación si existe.
3. Tomar las **3-5 más recientes por `scheduled_for`** del frontmatter de cada publicación (fecha de publicación, no de edición). Mostrarlas como contexto:

```
Aprendizajes recientes del pilar `<pillar-slug>`:
- "El hook con dato concreto funcionó mejor que el ángulo personal." (cardio-rmssd-newsletter, 2026-04-02)
- "Subject line tenía 53 chars — algunos clientes lo cortaron." (cardio-rmssd-newsletter, 2026-04-02)
- "Una respuesta tardía: @user preguntó si esto aplica a Zone 2." (cardio-rmssd-newsletter, 2026-04-02)
```

Por qué `scheduled_for` y no `mtime`: las publicaciones reciben aprendizajes a lo largo del tiempo (3b en weekly normal, paso 6 en cleanup) — eso modifica el mtime sin cambiar la fecha de publicación. Ordenar por mtime mostraría "recientes" que son aprendizajes anotados retroactivamente sobre publicaciones antiguas. `scheduled_for` es la fecha real.

Si una publicación tiene aprendizajes pero **no tiene `scheduled_for`** (estado corrupto: aprendizajes sin status published consistente), incluirla al final del grupo y avisar al usuario ("(publicación sin scheduled_for; revisar)").

4. Preguntar al usuario:

> "¿refinar la descripción de este pilar basándote en estos aprendizajes? (refinar / dejar / ver más bullets)"

- **Refinar** → entra en el sub-interview del pilar (mantener / refinar / reescribir descripción) con los aprendizajes como contexto activo.
- **Dejar** → no toca la descripción; sigue al siguiente pilar.
- **Ver más bullets** → muestra hasta 10 bullets, después se vuelve a preguntar.

Si **no hay aprendizajes** registrados en ese pilar:

```
(no hay aprendizajes registrados todavía para este pilar)
```

Y el flujo sigue al sub-interview normal del pilar sin contexto extra. Nunca bloquea el interview.

**Huérfanos**: si una publicación con `## Aprendizajes` referencia una idea que ya no existe (movida a `ideas/archive/`, eliminada, o el slug fue renombrado) — entonces no se puede mapear al pilar y los bullets se pierden de las vistas agrupadas. Al final de la fase 3 de strategy, **listar los huérfanos** una sola vez como warning:

```
⚠ Publicaciones con aprendizajes pero idea no encontrada (huérfanas):
- projects/<slug>/publications/cardio-rmssd-newsletter.md → idea `cardio-rmssd` no existe (¿archivada o renombrada?)
```

No bloquea el flujo; solo informa.

**Importante**: el frontmatter de la idea (`pillar:`) y de la publicación referencian estos slugs. Cambiar un slug = inconsistencia con ideas previas. Si el usuario renombra un pilar, avisar de que las ideas/publicaciones existentes con ese pilar quedarán "huérfanas" hasta que se actualicen (no es un error de skill, es una decisión del usuario).

#### 3.4 · Objetivos

> No son OKRs (ver "Conceptos clave · Objetivos ≠ OKRs"). Son intenciones con número: `<periodo>: <métrica concreta>`. Opcional dejarlo en `_pendiente_`.

Pregunta guía: "¿Qué quieres conseguir con este pilar de contenido en el siguiente trimestre / semestre?"

Aceptar lista libre. Sugerencias de formato:
- "Q3 2026: X publicaciones en newsletter, Y suscriptores nuevos."
- "H1 2027: 5 piezas long-form en blog, posicionarse para keyword X."

Si el usuario no tiene objetivos claros: dejar la sección con un placeholder (`_pendiente_`) y avisar.

#### 3.5 · Anti-temas

Pregunta guía: "¿Qué NO vas a escribir, aunque venga la tentación?"

Sugerencias:
- Tipos de hook que rechazas ("listas '10 razones por las que...'").
- Áreas temáticas explícitamente fuera (ej. "nada de macroeconomía", "nada de política").
- Formatos que no encajan con la voz (ej. "preguntas retóricas como apertura").

Anti-temas son frontera, no aspiración. Si el usuario duda, dejar vacío en lugar de inventar.

### Paso 4 · Construir el contenido

Componer `projects/<slug>/strategy.md`:

```markdown
---
slug: <slug>
last_reviewed: <YYYY-MM-DD de hoy>
pillars:
  - <pillar-slug-1>
  - <pillar-slug-2>
  - <pillar-slug-3>
---

## Posicionamiento

<contenido del bloque 3.1>

## Audiencia / ICP

<contenido del bloque 3.2>

## Pilares

### <pillar-slug-1>
<descripción>

### <pillar-slug-2>
<descripción>

### <pillar-slug-3>
<descripción>

## Objetivos

- <objetivo 1>
- <objetivo 2>

## Anti-temas

- <anti-tema 1>
- <anti-tema 2>
```

### Paso 5 · Mostrar diff y confirmar

- Modo create: mostrar el contenido completo y pedir confirmación antes de escribir.
- Modo update: mostrar diff (qué bloques cambiaron, qué se mantiene). Confirmar antes de escribir.

### Paso 6 · Escribir y actualizar `last_reviewed`

- Escribir `projects/<slug>/strategy.md`.
- `last_reviewed: <hoy>` siempre se actualiza tras una edición confirmada, incluso si solo se tocó un bloque.

### Paso 7 · Resumen final

Mostrar al usuario:

- Path del fichero.
- Resumen de qué bloques se tocaron (en modo update).
- `last_reviewed` actualizado a hoy.
- Recordatorio de coherencia: si cambió la lista de `pillars`, sugerir revisar ideas/publicaciones existentes que usen pilares ahora obsoletos.

## Escritura · Ficheros creados o modificados

| Fichero | Acción |
|---|---|
| `projects/<slug>/strategy.md` | Crear (modo create) o reescribir (modo update) tras confirmación |

## Criterios de éxito

- `strategy.md` existe con frontmatter (`slug`, `pillars`, `last_reviewed: hoy`).
- Las 5 secciones (`Posicionamiento`, `Audiencia / ICP`, `Pilares`, `Objetivos`, `Anti-temas`) están presentes (aunque alguna esté con `_pendiente_`).
- Cada pilar listado en el frontmatter tiene su correspondiente `### <slug>` en la sección Pilares con descripción.
- El usuario confirmó la escritura.

## Errores y casos límite

- **No existe el proyecto** (`projects/<slug>/` ausente): abortar, sugerir `/audience-ops-init <slug>` antes.
- **`strategy.md` está corrupto** (frontmatter inválido, etc.): mostrar lo que se ve, ofrecer reescribir de cero (modo create efectivo) o que el usuario lo arregle a mano.
- **Pilar slug colisiona** con uno existente: si el usuario añade un pilar nuevo con el mismo slug que otro, advertir y pedir desambiguar.
- **El usuario quiere dejar todo en `_pendiente_`**: permitir, pero advertir que `idea --promote` y `draft` tratarán como "sin restricciones" — la skill no falla, el usuario sabe que va a ciegas.
- **Objetivos con fechas pasadas**: avisar (probablemente quiere refrescarlos) pero permitir.
- **Más de 5 pilares**: advertir ("el contenido se diluye") pero no impedir.

## Principios que aplica

- **Cero magia.** Confirmación antes de escribir. Diff visible en modo update.
- **Una sola fuente de verdad por cosa.** Los pilares viven en el frontmatter de `strategy.md` (canónico) + descripción en el cuerpo. Las ideas referencian por slug.
- **Frontmatter mínimo.** Solo `slug`, `pillars`, `last_reviewed`. El resto en prosa con H2s.
- **El proyecto se deriva del path.** El fichero acaba en `projects/<slug>/`, no se duplica.
- **Soft archive desde el inicio.** Si el usuario pide "olvidar" la estrategia entera (poco común), mover a `projects/<slug>/archive/strategy-YYYY-MM-DD.md` siguiendo la convención `archive/` del repo, en lugar de borrar o renombrar adyacente.

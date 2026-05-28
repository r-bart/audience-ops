---
name: audience-ops-strategy
description: Interview para crear o actualizar el `strategy.md` de un proyecto (posicionamiento, ICP, pilares, objetivos, anti-temas). Avisa si `last_reviewed` lleva más del umbral configurado.
metadata:
  author: r-bart
  version: "0.10.0"
---

# strategy — Crear o actualizar la estrategia de un proyecto

## Cuándo usar esta skill

- El proyecto no tiene `strategy.md` aún (o solo tiene el placeholder que dejó `init` en modo bootstrap temprano).
- Quieres revisar / actualizar una estrategia existente: el contexto cambió, los pilares ya no encajan, los objetivos del trimestre son otros.
- `weekly --cleanup` te avisó de que `last_reviewed` lleva más de `strategy_review_months` (default 4) sin tocarse.

Esta skill **no toca** `voice.md` ni los canales — son ficheros separados.

## Entradas

- Opcional: slug del proyecto. Si no se da, usar `defaults.project` de `config.yaml`. Si tampoco, preguntar mostrando los activos de `portfolio.yaml`.
- Opcional: bloque concreto a revisar (`positioning`, `icp`, `pillars`, `goals`, `anti-topics`). Si no se da, recorrer todos.

## Lectura previa

Antes de actuar, leer:

1. `config.yaml` — para `defaults.project` y `strategy_review_months`.
2. `portfolio.yaml` — para validar que el proyecto existe y leer su `one_liner` (contexto inicial del interview).
3. `projects/<slug>/strategy.md` — si existe, para modo update.

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

**Importante**: el frontmatter de la idea (`pillar:`) y de la publicación referencian estos slugs. Cambiar un slug = inconsistencia con ideas previas. Si el usuario renombra un pilar, avisar de que las ideas/publicaciones existentes con ese pilar quedarán "huérfanas" hasta que se actualicen (no es un error de skill, es una decisión del usuario).

#### 3.4 · Objetivos

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
- **Soft archive desde el inicio.** Si el usuario pide "olvidar" la estrategia entera (poco común), mover a `projects/<slug>/strategy.archived-YYYY-MM-DD.md` en lugar de borrar.

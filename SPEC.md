# Audience Ops — Plan Final (MVP)

> Sistema operativo de contenidos para indie hackers: ficheros estructurados + skills de Claude que vehiculan el workflow de estrategia → idea → calendario → publicación, multicanal y multi-proyecto. La publicación se delega a herramientas de terceros (Typefully, Buffer, Ghost, etc.).

---

## 1. Propuesta de valor

**Problema.** Un indie hacker típico mantiene 2–5 proyectos y publica en 3–6 canales. El trabajo *alrededor* de publicar —estrategia, calendario, repurposing, captura de ideas, voz— vive en notas dispersas, Notion sobreorganizado o en la cabeza. Resultado: publicación inconsistente, mezcla de tonos entre proyectos, ideas perdidas, dependencia de la inspiración.

**Por qué las herramientas existentes no resuelven esto.**
- **Schedulers (Typefully, Buffer, Hypefury):** resuelven *publicar*, no *pensar*. Asumen que ya sabes qué decir.
- **Suites de marketing (HubSpot, ContentStudio):** pesadas, caras, para equipos.
- **Notion/Obsidian:** lienzo, no workflow. Cada uno inventa su sistema y lo abandona.

**Apuesta.** Capa de *pensamiento* en ficheros markdown/yaml, operada vía skills de Claude. Sin app, sin servidor. Publicación delegada porque ahí no hay valor diferencial.

**Para quién.** Indie hackers, makers y agencias de 1–3 personas que ya trabajan con ficheros y Claude Code, y gestionan más de un proyecto.

**Por qué ficheros + skills, no una app.**
- Portable: vive en tu repo, viaja contigo, se versiona con git.
- Componible: cualquier herramienta que lea markdown se enchufa.
- Sin lock-in: si Audience Ops muere mañana, tus ficheros siguen sirviendo.
- Skills son la UI: la conversación es la interfaz; no hay que diseñar app.

---

## 2. Alcance del MVP

**Dentro.**
- Estructura de ficheros canónica.
- 5 skills que cubren el workflow.
- Multi-proyecto y multi-canal desde el día 1.
- Newsletter como canal de primera clase (junto a X, LinkedIn, blog).
- Hygiene continua + cleanup trimestral integrados en una sola skill (`weekly`).
- Workflow hasta *publicación lista para enviar* — el draft queda en un fichero que el usuario copia/pega en su scheduler.

**Fuera.**
- Publicación automática a redes.
- Analítica post-publicación.
- Generación de imágenes/video (existen skills aparte que se referencian).
- Colaboración multi-usuario.
- UI web propia.
- CLI propio (se evalúa en v2).

---

## 3. Principios

1. **Filesystem como base de datos.** Sin SQLite, sin servidor. Todo markdown/yaml legible.
2. **Skills como workflow.** No hay app; la conversación con Claude es la UI.
3. **Una sola fuente de verdad por cosa.** Estado en frontmatter, no en carpetas ni en ficheros derivables.
4. **Vistas regenerables.** Calendario, índices, hygiene — se computan al vuelo.
5. **Convención sobre configuración.** Una sola forma canónica de estructurar cada cosa.
6. **El proyecto se deriva del path.** No se duplica en frontmatter.
7. **Single-instance por repo.** Cada repo de producto (o directorio dedicado) tiene su propia instancia de Audience Ops bajo `<repo>/audience-ops/`. Multi-proyecto se logra **teniendo el tool en varios repos**, no agregando proyectos dentro de un mismo `audience-ops/`. Si necesitas vistas cruzadas entre instancias, se hace fuera de la skill. Decisión consciente: simplificar el modelo común a costa de las superpowers cross-instance.
8. **Soft archive, nunca delete.** Mover a `archive/`, jamás `rm`. Git refuerza pero el archive baja la fricción mental.
9. **Cero magia.** Nada se mueve, archiva o publica sin confirmación del usuario.
10. **El usuario es dueño.** Texto plano, cualquier editor, git friendly.
11. **Agent-agnóstico.** Las skills son playbooks markdown legibles por cualquier agente (Claude Code, Cursor, Aider, o un humano siguiendo el script). Claude Code añade autodescubrimiento e invocación cómoda, pero la lógica vive en markdown puro, sin acoplamiento al motor.

---

## 4. Estructura de carpetas

```
<host-repo>/                              ← repo del producto o directorio dedicado
└── audience-ops/                         ← namespace fijo, una instancia por repo
    ├── config.yaml                       ← comportamiento (umbrales, flags); sin defaults.project
    ├── strategy.md                       ← positioning, ICP, pilares, objetivos, anti-temas
    ├── voice.md                          ← voz y tono
    ├── channels/
    │   ├── <id>.md                       ← un fichero por canal activo
    │   └── archive/                      ← canales pausados/retirados
    ├── ideas/
    │   ├── _inbox.md                     ← captura rápida, append-only, líneas con fecha
    │   ├── <slug>.md                     ← idea formalizada
    │   └── archive/                      ← ideas killed o abandonadas
    └── publications/
        ├── <idea-slug>-<channel>.md
        └── archive/
            ├── <año>/                    ← published > cleanup_threshold_days
            └── abandoned/                ← drafts muertos
```

El **repo de la herramienta** (este repo, `r-bart/audience-ops`) tiene su propio layout (README, AGENTS, SPEC, LICENSE, `.claude/skills/audience-ops-*/SKILL.md`, `skills.sh.json`) — es ortogonal a esta sección. Lo que se describe aquí es la estructura que `init` crea en el repo del usuario.

**Convenciones transversales.**
- `archive/` es válida en cualquier nivel dentro de `audience-ops/`. Las skills la ignoran en vistas activas.
- Filenames de publicaciones: `<idea-slug>-<channel>.md`. Sin prefijo de fecha (sort por frontmatter `scheduled_for`).
- Backlinks entre ideas: `[[idea-slug]]` en el cuerpo del markdown. Sin frontmatter, sin esquema.
- Las entradas del `_inbox.md` empiezan con `YYYY-MM-DD ·` para que `weekly` detecte staleness.

**El "proyecto" es implícitamente el repo host.** El identificador del proyecto, cuando se necesita en outputs (calendario, prompts, confirmaciones), se deriva del nombre del directorio raíz del repo (`basename(<repo-root>)`). No hay slug, no hay `name:` configurable. Multi-proyecto se logra teniendo el tool instalado en varios repos.

---

## 5. Esquemas de ficheros clave

### `audience-ops/config.yaml`

```yaml
owner:
  name: Roberto
  email: robertodzbt@gmail.com

behavior:
  inbox_auto_promote: false              # las ideas no se promueven solas
  cleanup_threshold_days: 90             # umbral para archivar publicaciones publicadas
  stale_draft_days: 30                   # weekly flagea drafts más viejos sin tocar
  stale_inbox_days: 30                   # weekly flagea entradas de inbox más viejas
  strategy_review_months: 4              # strategy avisa si last_reviewed es más viejo
  paused_archive_threshold_months: 6     # vestigial post-v0.12.0; conservado por compatibilidad
  learnings_prompt_threshold_days: 60    # weekly --cleanup ofrece añadir aprendizajes retroactivos
  prompt_learnings_on_publish: true      # weekly hygiene 3b pregunta por aprendizajes
  scheduler: typefully                   # hint informativo para el handoff ready → published
```

Sin bloque `defaults:` (eliminado en v0.12.0). El `scheduler` se conserva como hint dentro de `behavior:`. Sin `defaults.project` — el "proyecto" es implícitamente el repo host.

### `audience-ops/strategy.md`

```markdown
---
last_reviewed: 2026-04-12       # opcional; strategy avisa si pasa de strategy_review_months
pillars:
  - behind-the-build
  - fitness-data-thinking
  - lessons-from-shipping
---

## Posicionamiento
...

## Audiencia / ICP
...

## Pilares
### behind-the-build
...
### fitness-data-thinking
...

## Objetivos
- Q3 2026: ...

## Anti-temas
- ...
```

Frontmatter minimalista: solo lo que las skills necesitan parsear (`pillars`, `last_reviewed`). El resto es prosa con H2s convencionales.

### `audience-ops/voice.md`

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

### `audience-ops/channels/<id>.md`

```markdown
---
id: x
handle: "@verxion"
url: https://x.com/verxion
cadence: 3/week
status: active
---

## Formato
- Threads de 5–7 tweets máximo
- Primer tweet = hook (claim concreto, no pregunta retórica)
- Una idea por tweet, sin numerar

## Ajustes de voz (override sobre voice.md)
- Más contundente en el hook
- Permitido el "yo" en primera persona

## Sí
- Capturas del producto en uso
- Datos crudos sobre cardio/training

## No
- Hilos motivacionales genéricos
```

### `audience-ops/channels/newsletter.md`

```markdown
---
id: newsletter
handle: "verxion.com/newsletter"
platform: beehiiv
cadence: 1/week
status: active
---

## Formato
- Subject line: ≤50 chars, sin clickbait, promete una cosa concreta
- Preheader: complementa al subject, no lo repite
- Estructura: hook (2-3 frases) → cuerpo (1 idea principal) → cierre con CTA opcional
- Longitud: 400–700 palabras

## Anatomía de la publicación
El fichero `publications/<idea>-newsletter.md` debe llevar en frontmatter:
- `subject`
- `preheader` (opcional)

## Subject line
- Sí: "Cómo medí HRV durante 30 días"
- No: "🚀 Las 5 razones por las que..."

## Ajustes de voz
- Más cercano que en blog; "yo" y "tú" permitidos
```

### `audience-ops/ideas/_inbox.md`

```markdown
# Inbox

2026-05-21 · Hook sobre HRV en zona 1
2026-05-19 · Comparar Garmin vs Whoop para zona 2
2026-04-12 · Idea: la trampa del "10k pasos"     ← weekly flagea: >30 días
```

Una línea = una idea. Prefijo `YYYY-MM-DD ·`. Append-only.

### `audience-ops/ideas/<slug>.md`

```markdown
---
slug: why-i-killed-feature-x
pillar: lessons-from-shipping
channels: [blog, x]              # canales destino previstos
created: 2026-05-15
---

## Hook / ángulo
...

## Notas / research
...

## Referencias
- [[other-idea-slug]]            # backlink a otra idea
- https://...
```

Sin campo `status`. El estado se deriva del estado agregado de sus publicaciones.

### `audience-ops/publications/<idea-slug>-<channel>.md`

```markdown
---
idea: cardio-rmssd
channel: newsletter
status: ready                    # draft | ready | published
scheduled_for: 2026-05-28
subject: "El cardio que no estabas midiendo"
preheader: "30 días midiendo HRV en zonas suaves cambiaron mi semana de entrenos."
---

[contenido en el formato del canal]
```

Estado vive solo aquí. El proyecto se deriva del path.

### `## Aprendizajes` (opcional, post-publicación)

Cuando una publicación llega a `status: published`, puede acumular una sección opcional `## Aprendizajes` al final del cuerpo, con bullets libres:

```markdown
[cuerpo de la publicación]

## Aprendizajes

- El hook con dato concreto funcionó mejor que el ángulo personal.
- Subject line tenía 53 chars — algunos clientes lo cortaron en preview.
- Una respuesta nominal: "@user" preguntó si esto aplica a Zone 2.
```

Reglas:
- **Estructura libre**: bullets en plain text, sin schema. Se valoran "qué funcionó / qué no / para la próxima" pero la forma exacta queda al usuario.
- **Opcional**: una publicación published sin `## Aprendizajes` es válida.
- **Append-only en la práctica**: añade bullets a lo largo del tiempo si emergen nuevos aprendizajes (ej. una respuesta tardía).
- **Lo leen**: `weekly` (para hygiene retroactiva en cleanup) y `strategy` (para surface y posible iteración de pilares en modo update).
- **No es analítica**: aprendizajes son cualitativos. Engagement / open-rate / subscribers siguen out-of-scope per §2.

---

## 6. Skills MVP (5)

Todas las rutas son relativas a la instancia (`<repo-host>/audience-ops/`).

| Skill | Rol | Lee | Escribe |
|---|---|---|---|
| `init` | Bootstrap de una instancia (crea `audience-ops/` con strategy + voice + canales + inbox vacío) | — | toda la estructura `audience-ops/` |
| `strategy` | Interview para crear/actualizar `strategy.md`. Avisa si `last_reviewed` pasa del umbral. En modo update, lee aprendizajes por pilar para sugerir iteración | `audience-ops/strategy.md`, `audience-ops/publications/*.md` (aprendizajes) | `audience-ops/strategy.md` |
| `idea` | Dos modos: captura rápida al `_inbox` con prefijo de fecha; o promoción a idea estructurada (slug, pilar, canales, ángulo) | `audience-ops/strategy.md` (pilares), `audience-ops/ideas/_inbox.md` | `audience-ops/ideas/_inbox.md`, `audience-ops/ideas/<slug>.md` |
| `draft` | Idea + canal → draft en `audience-ops/publications/`. Modos `--from` (repurpose) y `--to` (batch a N canales). Incluye revisión final antes de marcar `ready` | `audience-ops/voice.md`, `audience-ops/channels/<id>.md`, idea (idea-slug obligatorio; anchor para repurpose), publicación existente (si repurpose) | `audience-ops/publications/<...>.md` |
| `weekly` | Ritual de la instancia local: triage del inbox, vista de calendario, qué promover a draft, hygiene continua. Modo `--cleanup` para limpieza trimestral | todo en `audience-ops/` | mueve estados en frontmatter, archiva |

### Detalle de `weekly` (la skill más cargada)

**Modo normal (cada semana).**
1. Triage de `_inbox.md`: para cada entrada nueva, ¿promover, dejar, matar?
2. Calendario: vista de esta instancia (sin cross-instance; no hay `weekly <proyecto>` argumento).
3. Hygiene continua:
   - Drafts > 30 días sin tocar → ¿abandonar?
   - Ready con `scheduled_for` ya pasado → ¿marcar como published?
   - Entradas de `_inbox` > 30 días → ¿promover o matar?
4. Promociones: qué ideas pasan a draft esta semana, en qué canales.

**Modo cleanup (`weekly --cleanup`, trimestral).**
1. Publicaciones published con `scheduled_for` > 90 días → mover a `publications/archive/<año>/`.
2. Drafts abandonados → mover a `publications/archive/abandoned/`.
3. Ideas killed → mover a `ideas/archive/`.
4. `strategy.md` con `last_reviewed` > 4 meses → propone revisión.
5. (Eliminado en v0.12.0: la propuesta de archivar proyectos pausados vivía en `portfolio.yaml`, que ya no existe).

Toda acción requiere confirmación. Nada se mueve solo.

---

## 7. Workflow típico

### Onboarding (una vez)

```
co:init
  → crea estructura raíz + primer proyecto
  → encadena strategy (interview) → voice → channels (define los activos)
```

### Día a día — captura

```
co:idea "hook sobre HRV en zona 1"
  → modo rápido: append a _inbox.md con fecha de hoy
```

### Promoción de inbox a idea estructurada

```
co:idea <texto del inbox>
  → modo estructurado: crea ideas/<slug>.md con frontmatter, pilar, canales previstos
```

### Drafting

```
co:draft <idea-slug> <channel>
  → lee voice.md + channels/<channel>.md + idea
  → produce draft en publications/<idea>-<channel>.md (status: draft)
  → al final, revisión guiada; si pasa, status → ready
```

### Repurposing

```
co:draft <idea-slug> <new-channel> --from <existing-publication>
  → adapta una publicación existente a otro canal
```

### Ritual semanal

```
co:weekly
  → triage inbox + calendario global + hygiene + decisiones de drafting
```

### Publicación (manual)

El usuario abre `publications/<...>.md` con status `ready`, copia el contenido en Typefully (o el scheduler configurado), programa. En el próximo `weekly` confirma el cambio de estado a `published`.

### Limpieza trimestral

```
co:weekly --cleanup
  → archivado de publicaciones viejas, drafts muertos, ideas killed, proyectos pausados; revisión de estrategia
```

---

## 8. Hygiene y limpieza

**Capas de hygiene:**

| Capa | Frecuencia | Mecanismo |
|---|---|---|
| Triage de inbox y drafts | Semanal | `weekly` modo normal |
| Marcado de publicaciones realizadas | Semanal | `weekly` modo normal |
| Archivado de published viejas | Trimestral | `weekly --cleanup` |
| Archivado de drafts/ideas abandonadas | Trimestral | `weekly --cleanup` |
| Revisión de estrategia | ~4 meses | `weekly --cleanup` o `strategy` al detectar `last_reviewed` viejo |
| Limpieza de proyectos pausados | Cuando ocurre | `weekly --cleanup` propone archivar |

**Reglas:**
- Mover, nunca borrar.
- Las vistas activas excluyen `archive/`.
- Cero limpieza automática sin confirmación.
- Sin cron, sin programación en background.

---

## 9. Distribución y compatibilidad

**Forma de distribución: repo template clonable.** El usuario clona el repo y obtiene en un solo paso la estructura + las skills. Cero instalación global.

**Compatibilidad con otros agentes.** Las 5 skills son **playbooks markdown** ubicados en `.claude/skills/<slug>/SKILL.md`. Cualquier agente (Claude Code, Cursor, Aider, otro Claude por API, o un humano leyendo) puede operar el sistema siguiendo el playbook correspondiente. Nada en la lógica de las skills depende de Claude Code; el path bajo `.claude/skills/` se elige porque Claude Code los descubre de ahí automáticamente, pero el contenido es markdown puro.

Cómo entra cada motor en el sistema:

| Motor | Cómo invoca las skills |
|---|---|
| **Claude Code** | Descubre automáticamente cada `SKILL.md` en `.claude/skills/`. Invocación natural: `/audience-ops-init`, `/audience-ops-draft`, etc. (kebab-case; la sintaxis `<plugin>:<skill>` está reservada a skills empaquetadas como plugin). |
| **Cursor / Aider / cualquier otro** | El usuario apunta al fichero: "sigue las instrucciones de `.claude/skills/audience-ops-draft/SKILL.md` con esta idea y este canal". |
| **API directo** | El playbook se pasa como system prompt o instrucción. |

`AGENTS.md` en raíz documenta esto para que cualquier agente que aterrice en el repo entienda cómo usarlo sin asumir Claude Code.

**Schedulers.** El `draft` produce formato **solo por canal**, no por scheduler. El scheduler vive en `config.yaml` como hint informativo. Si un scheduler concreto requiere marcadores específicos (e.g., separadores de Typefully), se anotan a mano en `channels/<id>.md`. Cero código.

---

## 10. Próximos pasos sugeridos

**Cambios arquitecturales (v0.12.0)**:

- Multi-proyecto deja de ser nativo del sistema. Single-instance por repo bajo `audience-ops/`.
- `portfolio.yaml` y `projects/<slug>/` desaparecen del modelo.
- Multi-proyecto se logra teniendo el tool instalado en varios repos.
- Cross-instance features (calendario global, repurpose cross-project) están explícitamente fuera; se retomarán si dogfooding las pide.

**Pendiente**:

1. **Dogfooding sobre un proyecto real** (Verxion u otro). Empezar.
2. **v1.0.0** se alcanza cuando dogfooding valide el MVP sin fricciones críticas pendientes.

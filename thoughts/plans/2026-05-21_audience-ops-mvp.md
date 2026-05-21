# Implementation Plan: Audience Ops MVP

**Date**: 2026-05-21
**Status**: Draft
**Spec**: `/SPEC.md`

---

## Overview

Construir el MVP de Audience Ops: un repo template clonable que combina (a) estructura de ficheros canónica para gestionar contenidos multicanal y multi-proyecto y (b) 5 playbooks markdown (skills) agent-agnósticos que vehiculan el workflow estrategia → idea → draft → publicación, con publicación delegada y limpieza integrada.

No hay código ejecutable: el MVP entero es markdown, yaml, y un `settings.json`. La "implementación" es escribir los playbooks de forma que cualquier agente (Claude Code, Cursor, Aider, etc.) pueda operarlos sin ambigüedad.

---

## Requirements

Trazadas desde el SPEC:

- [ ] Estructura raíz: `README.md`, `AGENTS.md`, `portfolio.yaml`, `config.yaml`, `projects/`, `skills/`, `.claude/settings.json`.
- [ ] Multi-proyecto desde día 1 (carpeta `projects/<slug>/` aun con un proyecto).
- [ ] Esqueleto de proyecto: `strategy.md`, `voice.md`, `channels/`, `ideas/`, `publications/`.
- [ ] 5 skills MVP: `init`, `strategy`, `idea`, `draft`, `weekly`.
- [ ] Skills escritas como playbooks markdown puros; agent-agnósticas.
- [ ] Compatibilidad Claude Code vía `.claude/settings.json` (descubrimiento de `skills/`).
- [ ] Soporte de newsletter como canal de primera clase (subject + preheader).
- [ ] Vista de calendario global y per-project, computada al vuelo desde `scheduled_for`.
- [ ] Convención `archive/` válida en cualquier nivel; vistas activas la ignoran.
- [ ] Convenciones transversales: backlinks `[[idea-slug]]`, `YYYY-MM-DD ·` en inbox, `last_reviewed:` opcional en `strategy.md`.
- [ ] `weekly` cubre triage + calendario + hygiene continua + modo `--cleanup` trimestral.

---

## Approach Analysis

### Option A: Top-down secuencial, value-driven

**Descripción.** Bootstrap → `init` → flujo mínimo de valor (`idea` + `draft`) → profundizar (`strategy`) → cerrar el ritual (`weekly`). Cada fase entrega algo dogfoodable.

**Pros**:
- Cada fase produce valor incremental.
- `init` se valida pronto (define el modelo de datos).
- El loop más corto de valor (captura → draft listo) llega en fase 3.
- Permite parar y dogfoodar entre fases sin estar a medias.

**Cons**:
- `init` necesita asumir cosas sobre skills aún no escritas (`strategy`). Mitigación: en fase 2, `init` llama a una versión rudimentaria *inline* de strategy; fase 5 la profundiza.

**Complexity**: Low–Medium

### Option B: Skeletons-first

**Descripción.** Fase 1: esqueletos vacíos de las 5 skills. Fase 2–5: rellenar.

**Pros**: ves la forma completa pronto.

**Cons**: skeletons son dead weight; incentivan commitment temprano a interfaces que pueden cambiar; sin valor real hasta avanzado el plan.

**Complexity**: Medium

### Option C: Test-driven por dogfooding

**Descripción.** Implementar lo mínimo de `init`, usar Audience Ops a mano sobre un proyecto real, escribir las skills retrospectivamente desde lo que funcionó.

**Pros**: skills emergen de uso real, no de teoría.

**Cons**: alta sobrecarga inicial; lento; el SPEC ya capitaliza pensamiento previo.

**Complexity**: High

### Recomendación

**Opción A.** Encaja con el orden de los próximos pasos del SPEC y mantiene cada fase shippable.

---

## Files to Create/Modify

| File | Action | Purpose |
|------|--------|---------|
| `README.md` | Create | Intro humana, cómo arrancar, mapa de skills |
| `AGENTS.md` | Create | Cómo opera cualquier agente el sistema |
| `portfolio.yaml` | Create | Índice de proyectos (template inicial vacío + comentario) |
| `config.yaml` | Create | Defaults de comportamiento |
| `.claude/settings.json` | Create | Registrar `skills/` para descubrimiento de Claude Code |
| `.gitignore` | Create | Excluir cosas obvias (sistema, editor) |
| `projects/.gitkeep` | Create | Mantener carpeta vacía |
| `skills/init.md` | Create | Playbook bootstrap + primer proyecto |
| `skills/idea.md` | Create | Playbook captura + promoción de ideas |
| `skills/draft.md` | Create | Playbook idea + canal → draft |
| `skills/strategy.md` | Create | Playbook interview de estrategia |
| `skills/weekly.md` | Create | Playbook ritual + cleanup |

---

## Implementation Phases

### Phase 1: Bootstrap del repo

Crear estructura raíz, ficheros meta, y la pieza de glue para Claude Code. **Sin skills aún** — solo el contenedor.

#### Task 1.1: README.md
**File**: `README.md`

Contenido mínimo:
- Qué es Audience Ops (1-2 párrafos).
- Cómo arrancar: clonar el repo, ejecutar `init`.
- Mapa de las 5 skills.
- Enlace al SPEC.

#### Task 1.2: AGENTS.md
**File**: `AGENTS.md`

Contenido:
- Premisa: las skills son playbooks markdown en `skills/`.
- Cómo entra Claude Code (autodescubrimiento vía `.claude/settings.json`).
- Cómo entra cualquier otro agente (apuntar al fichero del playbook).
- Principio: nada en `skills/` depende del motor.

#### Task 1.3: portfolio.yaml y config.yaml templates
**Files**: `portfolio.yaml`, `config.yaml`

Skeletons con valores por defecto + comentarios que el usuario edita en `init`.

#### Task 1.4: .claude/settings.json
**File**: `.claude/settings.json`

Configurar Claude Code para descubrir skills en `skills/`. Forma esperada (verificar contra docs de Claude Code en implementación):
```json
{
  "skills": { "paths": ["./skills"] }
}
```

#### Task 1.5: .gitignore y projects/.gitkeep
**Files**: `.gitignore`, `projects/.gitkeep`

`.DS_Store`, `*.swp`, etc. en gitignore; `.gitkeep` para mantener carpeta vacía en git.

---

### Phase 2: skills/init.md

La skill más cargada — bootstrap completo de un primer proyecto.

#### Task 2.1: Playbook init
**File**: `skills/init.md`

Frontmatter (compatible con Claude Code):
```yaml
---
name: audience-ops:init
description: Bootstrap del repo + primer proyecto (estructura + strategy + voice + canales en flujo guiado)
---
```

Cuerpo del playbook:
- Detectar si el repo ya está inicializado (`portfolio.yaml` con proyectos vs vacío).
- Si vacío: interview del owner → escribe `portfolio.yaml` con owner.
- Crear primer proyecto: pedir slug, name, one-liner, status → escribe entrada en `portfolio.yaml` y crea `projects/<slug>/` + subcarpetas.
- Llamar a sub-flujo de estrategia (versión rudimentaria inline; fase 5 la profundiza con `skills/strategy.md`).
- Llamar a sub-flujo de voz (interview corta → escribe `voice.md`).
- Llamar a sub-flujo de canales (preguntar qué canales activa, para cada uno: handle, cadencia, formato → escribe `channels/<id>.md`).
- Resumen final: estructura creada, próximos pasos sugeridos.

---

### Phase 3: skills/idea.md y skills/draft.md (loop de valor)

El flujo más corto que produce valor publicable.

#### Task 3.1: Playbook idea
**File**: `skills/idea.md`

Dos modos:
- **Quick**: usuario pasa un texto corto → append a `projects/<slug>/ideas/_inbox.md` con prefijo de fecha de hoy.
- **Promote**: usuario pasa una entrada del inbox o un texto formal → crear `ideas/<slug>.md` con frontmatter (slug, pilar, canales, created), pedir ángulo/hook, marcar la línea del inbox como promovida (opcional: eliminarla o tacharla).

Determinación de proyecto: derivar de `config.yaml` (default) o flag explícito.

Lee `strategy.md` para sugerir pilares válidos.

#### Task 3.2: Playbook draft
**File**: `skills/draft.md`

Entrada: `<idea-slug>` + `<channel>` (+ opcional `--from <publication-path>` para repurpose).

Pasos del playbook:
1. Leer `projects/<slug>/voice.md`, `projects/<slug>/channels/<channel>.md`, `projects/<slug>/ideas/<idea-slug>.md`.
2. Si hay `--from`, leer también la publicación origen.
3. Generar draft respetando: voz + ajustes de voz del canal + reglas de formato del canal + ángulo de la idea.
4. Para newsletter, generar también `subject` + `preheader` en frontmatter.
5. Escribir `projects/<slug>/publications/<idea-slug>-<channel>.md` con `status: draft`.
6. Revisión guiada: hook, claridad, longitud, alineación con voz/pilar. Si pasa, `status: ready`.

---

### Phase 4: Dogfooding mid-plan

**No es una fase de código.** Antes de continuar con `strategy` y `weekly`, usar lo construido sobre un proyecto real (Verxion o el propio Audience Ops) durante ~1 semana. Capturar fricciones en un fichero `thoughts/dogfooding-log.md`.

#### Task 4.1: Sesión de dogfooding
- Crear un proyecto real con `init`.
- Capturar 5–10 ideas reales con `idea` (modo quick).
- Promover 2–3 a ideas estructuradas (`idea` modo promote).
- Generar 2–3 drafts (`draft`).
- Anotar fricciones, ambigüedades, cosas no cubiertas en `thoughts/dogfooding-log.md`.

#### Task 4.2: Ajustar skills según hallazgos
- Editar `init.md`, `idea.md`, `draft.md` con las correcciones aprendidas.
- Si los hallazgos afectan al SPEC, actualizarlo también.

---

### Phase 5: skills/strategy.md

Interview profundo para crear/actualizar `strategy.md` de un proyecto.

#### Task 5.1: Playbook strategy
**File**: `skills/strategy.md`

Pasos:
1. Detectar si `strategy.md` ya existe en el proyecto target. Si sí, leerlo.
2. Si existe: comprobar `last_reviewed`; si pasa de 4 meses, avisar.
3. Interview por bloques (cada bloque se puede saltar si ya está bien):
   - Posicionamiento.
   - Audiencia / ICP.
   - Pilares (lista + descripción de cada uno).
   - Objetivos (trimestrales o lo que sea).
   - Anti-temas.
4. Escribir `strategy.md` con frontmatter (`slug`, `pillars`, `last_reviewed: hoy`) + prosa con H2s convencionales.
5. **No tocar `voice.md`** — es otro fichero.

Esto reemplaza el sub-flujo inline de estrategia que `init` traía en fase 2: `init.md` se actualiza para delegar en `strategy.md` cuando exista.

---

### Phase 6: skills/weekly.md

El ritual y la limpieza. La skill más amplia pero la última porque requiere datos acumulados.

#### Task 6.1: Playbook weekly modo normal
**File**: `skills/weekly.md`

Estructura del playbook:
1. **Triage del inbox**: leer `_inbox.md` de cada proyecto (o el activo). Para cada entrada: ¿promover, dejar, matar?
2. **Vista de calendario**: walk `projects/*/publications/*.md` (excluyendo `archive/`), parsear frontmatter, agrupar por semana / canal / proyecto. Render global por defecto; con argumento de proyecto → filtro.
3. **Hygiene continua**:
   - Drafts con mtime > `stale_draft_days` (de `config.yaml`) → preguntar.
   - Ready con `scheduled_for < hoy` → ¿marcar como published?
   - Entradas de `_inbox` con fecha > `stale_inbox_days` → preguntar.
4. **Decisiones de drafting**: ofrecer ideas en estado idea para promover a draft esta semana.

#### Task 6.2: Modo cleanup trimestral
Mismo fichero `skills/weekly.md`, con sección que se activa al invocar con `--cleanup`.

Acciones (todas con confirmación):
- Publicaciones published con `scheduled_for` > `cleanup_threshold_days` → mover a `publications/archive/<año>/`.
- Drafts abandonados → mover a `publications/archive/abandoned/`.
- Ideas killed → mover a `ideas/archive/`.
- `strategy.md` con `last_reviewed` viejo → proponer revisión.
- Proyectos `paused` desde hace tiempo en `portfolio.yaml` → proponer `archived`.

---

## Task Dependencies

```yaml
dependencies:
  1.1: []
  1.2: []
  1.3: []
  1.4: []
  1.5: []
  2.1: [1.1, 1.2, 1.3, 1.4, 1.5]   # init asume la estructura raíz definida
  3.1: [2.1]                        # idea opera sobre proyectos creados por init
  3.2: [2.1]                        # draft opera sobre proyectos + idea
  4.1: [3.1, 3.2]                   # dogfooding necesita idea + draft
  4.2: [4.1]                        # ajustes vienen del log
  5.1: [4.2]                        # strategy llega tras ajustes de dogfooding
  6.1: [3.1, 3.2, 5.1]              # weekly necesita el modelo entero
  6.2: [6.1]
```

Tareas de fase 1 son paralelizables entre sí (no hay dependencias intra-fase).

---

## Risk Analysis

### Edge Cases

- [ ] **Repo ya inicializado** cuando se invoca `init`: detectar y ofrecer modo "añadir proyecto" en lugar de bootstrap completo.
- [ ] **Slug colisiona** con proyecto existente: prompt para renombrar.
- [ ] **Canal referenciado en `draft` que no existe** en `channels/`: error claro + sugerir canales disponibles.
- [ ] **Idea sin pilar válido** según `strategy.md`: warning, permitir continuar.
- [ ] **Publicación con `scheduled_for` en el pasado y status `ready`**: hygiene de `weekly` lo flagea.
- [ ] **Newsletter sin `subject`** en frontmatter: `draft` lo exige; revisión falla si falta.
- [ ] **Inbox sin prefijo de fecha** en alguna línea: `weekly` ignora (no asume), avisa al usuario.

### Technical Risks

- [ ] **`.claude/settings.json` formato exacto**: la sintaxis para registrar paths de skills puede variar; verificar contra docs vigentes durante fase 1.
- [ ] **Compatibilidad con Cursor/Aider/otros**: los playbooks deben evitar lenguaje Claude-Code-específico ("usa el tool X", "invoca SkillTool"). Validar fase 1 con un agente alternativo aunque sea manualmente.
- [ ] **Tamaño de los playbooks**: si una skill markdown crece demasiado, agentes pequeños la procesan mal. Mantener cada `skills/<name>.md` <500 líneas; partir si fuera necesario (no en MVP).
- [ ] **Walk de publicaciones lento** si hay muchas: irrelevante en MVP; volúmenes pequeños.
- [ ] **Sincronización de estado tras edición manual**: el usuario puede editar frontmatter a mano y romper invariantes; `weekly` debe ser robusto a esto (no asumir formato perfecto).

---

## Testing Strategy

No hay tests automatizados — el MVP es markdown. La validación es **dogfooding + verificación manual** estructurada.

### Verificación manual por fase

| Fase | Cómo verificar |
|---|---|
| 1 | Clonar el repo desde cero, `ls -la`, abrir cada fichero, comprobar que estructura está completa y READMEs son legibles. |
| 2 | Invocar `init` en repo recién clonado; comprobar que crea `projects/<slug>/` con todas las subcarpetas, escribe `portfolio.yaml`, dispara los tres sub-flujos. |
| 3 | Invocar `idea` modo quick → `_inbox.md` actualizado. `idea` modo promote → fichero idea creado. `draft <idea> <channel>` → publicación con frontmatter correcto y contenido alineado a voz + formato. |
| 4 | Log de dogfooding existe y tiene ≥3 fricciones documentadas. |
| 5 | Invocar `strategy` sobre proyecto existente; resulta `strategy.md` con todas las secciones esperadas y `last_reviewed` de hoy. |
| 6 | Invocar `weekly`; resulta vista de calendario, triage del inbox, hygiene. Invocar `weekly --cleanup` con publicaciones >90d → mueve a archive/ tras confirmación. |

### Verificación con segundo agente

Tras fase 3, abrir `skills/draft.md` con Cursor o Aider y pedir "sigue las instrucciones de este playbook con esta idea y este canal". Si el agente alternativo puede ejecutarlo sin pedirme información específica de Claude Code, principio agent-agnóstico validado.

---

## Done Criteria

### Phase 1: Bootstrap
- [ ] `README.md` existe, ≤200 líneas, incluye mapa de skills y enlace al SPEC.
- [ ] `AGENTS.md` existe y documenta el modelo agent-agnóstico.
- [ ] `portfolio.yaml` y `config.yaml` están como templates con comentarios.
- [ ] `.claude/settings.json` registra `skills/` (verificar que Claude Code los detecta listando skills).
- [ ] `projects/` existe (vía `.gitkeep`) y `skills/` existe vacía.
- [ ] `.gitignore` cubre lo básico (DS_Store, swap, etc.).

### Phase 2: init
- [ ] `skills/init.md` existe con frontmatter compatible Claude Code.
- [ ] Invocar `init` en repo vacío produce: entrada en `portfolio.yaml`, `projects/<slug>/` con `strategy.md`, `voice.md`, `channels/<id>.md` (al menos uno), `ideas/_inbox.md` (vacío), `publications/` (vacía).
- [ ] Invocar `init` en repo con proyectos detecta y ofrece "añadir proyecto" en lugar de bootstrap.

### Phase 3: idea + draft
- [ ] `skills/idea.md` y `skills/draft.md` existen.
- [ ] `idea` modo quick añade línea con `YYYY-MM-DD ·` a `_inbox.md`.
- [ ] `idea` modo promote crea `ideas/<slug>.md` con frontmatter completo.
- [ ] `draft` produce publicación con frontmatter (`idea`, `channel`, `status: draft`, `scheduled_for`) y contenido que cumple las reglas del canal.
- [ ] `draft` con `--from` puede adaptar una publicación a otro canal.
- [ ] Revisión guiada al final de `draft` mueve a `status: ready` si pasa.

### Phase 4: Dogfooding
- [ ] `thoughts/dogfooding-log.md` documenta ≥3 fricciones reales.
- [ ] Cambios derivados aplicados a los playbooks (commits visibles tocando `skills/*.md`).

### Phase 5: strategy
- [ ] `skills/strategy.md` existe.
- [ ] Invocar `strategy` sobre proyecto existente actualiza `strategy.md` con frontmatter (`pillars`, `last_reviewed`) y todas las secciones (Posicionamiento / Audiencia / Pilares / Objetivos / Anti-temas).
- [ ] `init.md` actualizado para delegar en `strategy.md` cuando el sub-flujo aplique.

### Phase 6: weekly
- [ ] `skills/weekly.md` existe con modos normal y `--cleanup`.
- [ ] `weekly` (modo normal) renderiza calendario global, hace triage del inbox, surfacea hygiene (drafts viejos, ready vencidos, inbox stale).
- [ ] `weekly --cleanup` propone movimientos a `archive/` con confirmación y los aplica.
- [ ] Las vistas activas excluyen rutas `*/archive/*`.

### Overall
- [ ] Todas las fases completas.
- [ ] Un segundo agente (Cursor/Aider/otro) puede ejecutar al menos `skills/draft.md` siguiendo el playbook sin información extra.
- [ ] Sin TODOs ni placeholders en los playbooks finales.
- [ ] SPEC.md y plan en sintonía (cualquier desviación detectada en dogfooding aplicada a SPEC).

---

## Verification

No hay package manager — el repo es solo markdown/yaml/json. La verificación es estructural + dogfooding.

Comandos útiles durante el desarrollo:

```bash
# Estructura completa
find . -type f -not -path '*/\.*' -not -path '*/node_modules/*' | sort

# Lint mínimo de YAML (opcional, si se quiere automatizar)
# (no incluido en MVP)

# Verificación manual de skill discovery por Claude Code
# (invocar /audience-ops:init y comprobar que aparece)
```

Tras cada fase, ejecutar `/post-review` para checkpoint de requisitos cumplidos.

---

## Notas operativas

- **No usar prefijos de fecha en filenames de publicaciones** (consistente con SPEC §4).
- **Estado vive en frontmatter**, nunca en path (consistente con SPEC §3 principio 3).
- **El proyecto se deriva del path**, no se duplica (SPEC §3 principio 6).
- **Soft archive, never delete** (SPEC §3 principio 8).
- **Cero magia** — toda acción que escribe ficheros requiere confirmación del usuario (SPEC §3 principio 9).

Cualquier desviación de estos principios durante implementación debe (a) corregirse o (b) actualizarse en el SPEC con razón.

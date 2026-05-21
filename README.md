# Audience Ops

Sistema operativo de contenidos para indie hackers: ficheros estructurados + skills agent-agnósticas que vehiculan el workflow estrategia → idea → draft → publicación. Multicanal, multi-proyecto. La publicación se delega a terceros (Typefully, Buffer, Ghost, etc.).

> Para el detalle completo: ver [`SPEC.md`](./SPEC.md).
> Para cómo opera cualquier agente: ver [`AGENTS.md`](./AGENTS.md).

## Cómo arrancar

1. Clonar este repo en tu carpeta de trabajo.
2. Desde Claude Code (u otro agente compatible), invocar `/audience-ops:init`.
3. Seguir el flujo guiado: primer proyecto + estrategia + voz + canales.

## Mapa de skills (MVP)

| Skill | Para qué |
|---|---|
| `init` | Bootstrap del repo + primer proyecto. |
| `strategy` | Crear o actualizar la estrategia de un proyecto. |
| `idea` | Capturar ideas rápido al inbox, o promoverlas a ideas estructuradas. |
| `draft` | Convertir una idea en un draft para un canal concreto. Soporta repurpose. |
| `weekly` | Ritual semanal: triage, calendario, hygiene. Modo `--cleanup` para limpieza trimestral. |

## Estructura

```
audience-ops/
├── README.md            ← este fichero
├── AGENTS.md            ← cómo opera cualquier agente
├── SPEC.md              ← especificación completa
├── portfolio.yaml       ← qué proyectos existen
├── config.yaml          ← defaults de comportamiento
├── skills/              ← los 5 playbooks markdown
├── .claude/             ← glue para Claude Code
└── projects/            ← un subdirectorio por proyecto
```

Cada proyecto vive bajo `projects/<slug>/` con su propia estructura:

```
projects/<slug>/
├── strategy.md          ← posicionamiento, ICP, pilares, objetivos
├── voice.md             ← voz y tono
├── channels/            ← un fichero por canal activo
├── ideas/               ← repositorio de ideas
│   ├── _inbox.md        ← captura rápida
│   └── <slug>.md        ← ideas estructuradas
└── publications/        ← drafts, ready, published (estado en frontmatter)
```

## Principios clave

- **Filesystem como base de datos.** Sin servidor, sin app.
- **Skills como workflow.** La conversación es la UI.
- **Estado en frontmatter**, no en carpetas ni en ficheros derivables.
- **Vistas regenerables.** Calendario e índices se computan al vuelo.
- **El proyecto se deriva del path**, no se duplica en frontmatter.
- **Soft archive, never delete.** Mover a `archive/`, jamás `rm`.
- **Cero magia.** Toda acción que escribe ficheros requiere confirmación.

## Estado

MVP en construcción. Ver [`thoughts/plans/`](./thoughts/plans/) para el plan vigente.

# AGENTS.md

Audience Ops es agent-agnóstico. Las 5 skills viven en [`skills/`](./skills/) como playbooks markdown puros. Cualquier agente capaz de leer markdown puede operar el sistema.

## Cómo entra cada motor

### Claude Code

Las skills se descubren automáticamente vía [`.claude/settings.json`](./.claude/settings.json). Invocación natural:

```
/audience-ops:init
/audience-ops:idea "hook sobre HRV en zona 1"
/audience-ops:draft cardio-rmssd newsletter
/audience-ops:strategy
/audience-ops:weekly
```

### Otros agentes (Cursor, Aider, Codex, etc.)

Apuntar al fichero del playbook:

> "Sigue las instrucciones de `skills/draft.md` con la idea `cardio-rmssd` y el canal `newsletter`."

Los playbooks están escritos para ser **autocontenidos**: incluyen contexto, pasos a seguir, ficheros a leer, ficheros a escribir y criterios de éxito.

### API directo / scripts

Pasar el contenido del playbook como system prompt o como instrucción al modelo.

## Principios para agentes operando este repo

1. **Lee antes de escribir.** Las skills tocan ficheros del usuario. Siempre comprobar el estado actual antes de modificar.
2. **Confirma antes de mover.** Cualquier movimiento de ficheros (especialmente a `archive/`) requiere confirmación explícita del usuario.
3. **Sin magia silenciosa.** Si infieres algo no explícito (proyecto activo, canal por defecto, pilar más probable), dilo y pide confirmación.
4. **El proyecto se deriva del path**, no se duplica en frontmatter.
5. **Soft archive, never delete.** Mover a `archive/` siempre; nunca borrar.
6. **Una sola fuente de verdad por cosa.** Estado en frontmatter, no en carpetas. Nada que sea derivable se almacena.
7. **No introducir prefijos de fecha en filenames de publicaciones.** Sort por frontmatter (`scheduled_for`).

## Convenciones transversales

- Backlinks entre ideas: `[[idea-slug]]` en el cuerpo del markdown.
- Entradas del inbox: prefijo `YYYY-MM-DD ·` (una línea = una idea).
- Filenames de publicaciones: `<idea-slug>-<channel>.md`, sin prefijo de fecha.
- Estado de publicación: en frontmatter (`status: draft | ready | published`).
- Carpetas `archive/` válidas en cualquier nivel; vistas activas las ignoran.
- Frontmatter mínimo: solo campos que las skills necesitan parsear. El resto va en prosa con H2s convencionales.

## Anatomía de un playbook

Cada `skills/<name>.md` sigue esta estructura:

1. **Frontmatter**: `name`, `description` (compatible con Claude Code skill format).
2. **Cuándo usar**: en qué situación se invoca.
3. **Entradas**: qué necesita el agente del usuario o del filesystem.
4. **Lectura previa**: ficheros que el agente lee antes de actuar.
5. **Pasos**: secuencia de acciones, con puntos de confirmación.
6. **Escritura**: qué ficheros se crean o modifican y con qué forma.
7. **Criterios de éxito**: cómo verificar que la skill cumplió su objetivo.

Esto hace que un agente alternativo pueda seguir el playbook sin contexto adicional.

## Qué hace un agente cuando algo no está claro

- Si falta un fichero esperado: avisa, no asume.
- Si el frontmatter tiene un formato inesperado: muestra lo que ve, pide al usuario.
- Si una skill espera un proyecto y no está claro cuál: pregunta. No use el default sin avisar.
- Si el resultado de una acción es ambiguo: muestra el diff propuesto antes de escribirlo.

# Audience Ops

> A content operating system for indie hackers. Filesystem-first, agent-agnostic, multi-project, multi-channel. Publishing is delegated to tools you already use (Typefully, Buffer, Ghost, Beehiiv).

---

## Value proposition

Audience Ops is not a scheduler nor a marketing suite. It's the **thinking layer** missing between "I have an idea" and "hit publish":

- **Strategy, voice, and channels** defined per project.
- **Idea capture** with zero friction.
- **Drafting** that respects each channel's voice and format.
- **Calendar and weekly ritual** computed, not authored.
- **Periodic cleanup** built in.

Everything lives in markdown and yaml, inside a repo that's yours. No app, no server, no lock-in.

### Why existing tools don't solve this

| Tool | What it does | What it doesn't do |
|---|---|---|
| Typefully, Buffer, Hypefury | Schedule and publish | Help you think what to publish |
| HubSpot, ContentStudio | Full marketing suite for teams | Designed for 1–3 person operations |
| Notion, Obsidian | Canvas + database | Provide a workflow without reinventing it every time |

Audience Ops assumes you already have a scheduler (whichever you prefer) and focuses on what's missing: **the structured workflow between your head and the publish button**.

### Why filesystem + skills, not an app

- **Portable.** Lives in your repo, travels with you, versions in git.
- **Composable.** Anything that reads markdown plugs in.
- **No lock-in.** If Audience Ops disappears tomorrow, your files still work.
- **Agent-agnostic.** Today Claude Code, tomorrow something else. The logic lives in markdown any agent can read.
- **Skills as UI.** The conversation with an agent is the interface. No app to maintain or redesign.

---

## Who is this for?

- **Indie hackers, makers, and solopreneurs** running 2–5 projects in parallel.
- **Tiny agencies** (1–3 people) managing content for several clients.
- People comfortable working with **markdown**, **git**, and **AI agents** in their terminal or editor.

If you don't use markdown and don't have 2+ projects in flight, Notion or a scheduler is probably enough for you.

---

## Architecture

### Principles

1. **Filesystem as database.** Human-readable markdown and yaml.
2. **Skills as workflow.** No app; the conversation with the agent is the UI.
3. **One source of truth per thing.** State in frontmatter, never in folders or in derivable files.
4. **Regenerable views.** Calendar and indexes are computed on demand from data.
5. **Multi-project native.** Even with one project, it lives under `projects/<slug>/`.
6. **Project derived from path**, never duplicated in frontmatter.
7. **Soft archive, never delete.** Move to `archive/`, never `rm`.
8. **Zero magic.** Every action that writes files requires user confirmation.
9. **Agent-agnostic.** Skills are pure markdown playbooks; any markdown-capable agent can run them.

### Layout

**The skills** (installed via `npx skills add` — see [Install](#install)):

```
~/.claude/skills/                       ← global (with -g)
  or ./.claude/skills/                  ← per-project (without -g)
├── audience-ops-init/SKILL.md
├── audience-ops-idea/SKILL.md
├── audience-ops-draft/SKILL.md
├── audience-ops-strategy/SKILL.md
└── audience-ops-weekly/SKILL.md
```

**Your content directory** (created by `/audience-ops-init`, lives in a repo of yours):

```
your-content-repo/
├── portfolio.yaml        ← project index
├── config.yaml           ← behavior defaults
└── projects/
    └── <slug>/
        ├── strategy.md   ← positioning, ICP, pillars, goals
        ├── voice.md      ← voice and tone
        ├── channels/     ← one file per active channel (X, LinkedIn, newsletter, blog…)
        │   ├── x.md
        │   └── newsletter.md
        ├── ideas/
        │   ├── _inbox.md ← quick capture, append-only, dated lines
        │   └── <slug>.md ← structured ideas
        └── publications/ ← drafts + ready + published (state in frontmatter)
            └── <idea>-<channel>.md
```

The two are **separate by design**: the skills are the tool, your content is yours. Don't put content inside the audience-ops repo.

### Data model in 30 seconds

- **A project** has one `strategy.md`, one `voice.md`, N channels (one file each in `channels/`), an idea inbox, and publications per channel.
- **An idea** lives in a single file. It can spawn one or more publications (one per channel). No `status` field — the idea's state is derived from the aggregated state of its publications.
- **A publication** is `<idea>-<channel>.md`. Its state (`draft | ready | published`) lives in frontmatter. Scheduling = setting `scheduled_for:`.
- **The calendar** is a view computed on demand from all `scheduled_for` values across the repo. It is not a file.

---

## Features (5 MVP skills)

| Skill | What it does | Status |
|---|---|---|
| `init` | Bootstrap the repo + first project. Chains strategy, voice, and channels creation in a guided flow. | ✅ |
| `idea` | Two modes. **Quick**: capture one line into the project's inbox. **Promote**: turn an inbox entry into a structured idea (slug, pillar, channels, angle). | ✅ |
| `draft` | Takes an idea + a channel and produces a draft respecting voice, channel format, and angle. Supports repurpose (`--from`) to adapt an existing publication to a different channel. | ✅ |
| `strategy` | Interview to create or update a project's strategy: positioning, ICP, pillars, goals, anti-topics. Warns when `last_reviewed` is stale. | ✅ |
| `weekly` | Weekly ritual: inbox triage, calendar view (global or per-project), hygiene of stale drafts and overdue ready items. `--cleanup` mode for quarterly archive sweeps. | ✅ |

Skills are **self-contained markdown playbooks**. Each one documents inputs, prior reads, steps, writes, and success criteria.

### Newsletter as a first-class channel

Newsletter fits the same structure as any other channel: a file in `channels/newsletter.md` with format + rules. Newsletter publications carry `subject` and `preheader` in frontmatter, handled automatically by `draft`.

### Built-in cleanup

There's no separate `cleanup` skill — it lives as `weekly --cleanup`:

- Published items > 90 days → move to `publications/archive/<year>/`.
- Abandoned drafts → `publications/archive/abandoned/`.
- Killed ideas → `ideas/archive/`.
- Strategy with `last_reviewed` > 4 months → suggest review.
- Long-paused projects → suggest `archived` status.

All with confirmation. Zero automatic archiving.

---

## How to use it

### Install

Install the skills via [skills.sh](https://skills.sh) (works with Claude Code, Cursor, Codex, and 50+ agents):

```bash
# Global — recommended for multi-project use
npx skills add r-bart/audience-ops -g
```

Drop the `-g` if you want the skills committed alongside your content (installed to `./.claude/skills/` in the current directory, pinned to one version).

> **Reading the source or contributing?**
>
> ```bash
> git clone https://github.com/r-bart/audience-ops.git
> ```
>
> Don't use the clone as your content workspace — content lives in a separate repo (see [Layout](#layout)).

Audience Ops is designed primarily for use with **Claude Code**, but any markdown-capable agent can run the skills (see `AGENTS.md`).

### Day 1 · Bootstrap

Run this **in the directory that will hold your content** — a fresh repo of yours, not inside `audience-ops/`:

```
/audience-ops-init
```

Answer a short interview: your name, first project, positioning, voice, active channels. When done, you'll have:

- `portfolio.yaml` with your first project.
- `projects/<slug>/strategy.md`, `voice.md`, `channels/<id>.md` (one per channel).
- An empty idea inbox ready for capture.

### Day to day · Capture

```
/audience-ops-idea "hook about how I measured HRV for 30 days"
```

Quick mode: appends one line to the active project's `_inbox.md` with today's date. Zero friction.

### When an idea matures · Structure it

```
/audience-ops-idea --promote "the cardio you weren't measuring"
```

Promote mode: asks for pillar, target channels, hook, and creates a structured file at `ideas/<slug>.md`. The original inbox line gets marked with a `→ ideas/<slug>.md` suffix for traceability.

### Drafting · Idea + channel

```
/audience-ops-draft cardio-rmssd newsletter
```

Reads the project's voice, the newsletter channel rules, and the idea's angle. Generates a draft with `subject`, `preheader`, and body. Shows it to you, writes it on confirmation, then runs a review checklist: if it passes, marks `status: ready`; otherwise stays `draft` with notes on what's missing.

### Repurpose · Adapt to another channel

```
/audience-ops-draft cardio-rmssd x --from projects/verxion/publications/cardio-rmssd-newsletter.md
```

Adapts an existing publication to a different channel without losing the angle. Not copy-paste — reformats according to the destination channel's rules.

### Weekly ritual · Triage & calendar

```
/audience-ops-weekly
```

- Inbox triage: promote, leave, or kill?
- Global calendar view with slot conflicts and per-channel summary.
- Hygiene: stale drafts, overdue ready items, old inbox entries.
- Decide which ideas move to draft this week.

### Quarterly cleanup · Archive sweep

```
/audience-ops-weekly --cleanup
```

Proposes archiving for old published items, abandoned drafts, killed ideas, paused projects. All with confirmation.

### Publishing · Manual handoff

Audience Ops **does not publish**. When a publication reaches `status: ready`, open the file, copy the contents into Typefully (or whatever you use), schedule. On the next `weekly` you confirm the transition to `published`.

---

## Agent compatibility

Works with:

- **Claude Code** — skills autodiscovered from `.claude/skills/<slug>/SKILL.md`. Natural invocation with `/audience-ops-<skill>` (kebab-case; the `:` syntax is reserved for plugin-packaged skills).
- **Cursor / Aider / Codex / others** — point at the file: *"follow the instructions in `.claude/skills/audience-ops-draft/SKILL.md` with this idea and this channel"*.
- **Direct API** — pass the playbook as a system prompt.

See [`AGENTS.md`](./AGENTS.md) for details.

---

## Current status

**MVP shipped.** All 5 skills functional; v1.0.0 reserved for validation through real-content use.

- ✅ Phase 1 · Repo bootstrap.
- ✅ Phase 2 · `.claude/skills/audience-ops-init/SKILL.md`.
- ✅ Phase 3 · `.claude/skills/audience-ops-idea/SKILL.md` + `.claude/skills/audience-ops-draft/SKILL.md`.
- ✅ Phase 5 · `.claude/skills/audience-ops-strategy/SKILL.md`.
- ✅ Phase 6 · `.claude/skills/audience-ops-weekly/SKILL.md` (normal + `--cleanup` modes).

See [`thoughts/plans/2026-05-21_audience-ops-mvp.md`](./thoughts/plans/2026-05-21_audience-ops-mvp.md) for the full plan and [`SPEC.md`](./SPEC.md) for the technical spec.

---

## Contributing

This is a personal project under construction. If the model resonates and you want to adapt it to your workflow, fork freely. Real-usage issues welcome.

---

## License

MIT — see [`LICENSE`](./LICENSE).

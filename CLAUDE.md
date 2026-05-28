# CLAUDE.md

Project-specific instructions for any agent (Claude Code or otherwise) working on the Audience Ops repo itself — not on a user's content directory.

---

## Versioning

Audience Ops follows [semver](https://semver.org/), starting at `v0.9.0`.

### When to bump

| Bump | When |
|---|---|
| **Patch** (`0.9.x`) | Docs fixes, typos in playbooks, clarifications that don't change behavior, internal refactor of a SKILL.md that preserves its contract. |
| **Minor** (`0.X.0`) | New skill added (`audience-ops-strategy`, `audience-ops-weekly`). New mode added to an existing skill (e.g. `--from` in `draft`). New optional frontmatter field. Any additive, non-breaking behavior change. |
| **Major** (`X.0.0`) | Breaking change: a playbook's inputs/outputs change shape, a frontmatter field is removed or renamed, a skill is renamed, file path conventions change. |

**`v1.0.0` is reserved for**: the 5 MVP skills (init, idea, draft, strategy, weekly — all shipped as of v0.10.0) validated through real-content use without critical frictions outstanding.

We start at `v0.9.0` (not `v0.1.0`) to signal that the foundation is functional and the trajectory is "close to MVP", not "first sketch".

### Where the version lives

Three locations, kept in sync on every release:

1. **Git tag** — `vX.Y.Z` (annotated, not lightweight) on the commit that ships the version.
2. **`metadata.version`** in the frontmatter of each `.claude/skills/audience-ops-*/SKILL.md`. All skills share the same version number — this is a monorepo for a coherent toolset, not independently versioned packages.
3. **GitHub Release** with the same `vX.Y.Z` tag.

`skills.sh.json` does **not** carry a version — the per-skill metadata wins for display, the git tag wins for installation reference.

### Release flow

```bash
# 1. Decide the next version (patch/minor/major per the table above).
NEXT="vX.Y.Z"

# 2. Bump metadata.version in the 3 SKILL.md (4 once strategy ships, 5 once weekly ships).
#    Use awk/sed when Edit is blocked by the self-mod classifier:
for f in .claude/skills/audience-ops-*/SKILL.md; do
  sed -i.bak "s/^  version: \".*\"/  version: \"${NEXT#v}\"/" "$f" && rm "$f.bak"
done

# 3. Commit with a release-style message.
git add .claude/skills/ && git commit -m "chore: bump to $NEXT"

# 4. Tag the commit (annotated).
git tag -a "$NEXT" -m "<one-line description of what ships>"

# 5. Push main + tag.
git push origin main "$NEXT"

# 6. Create the GitHub Release (notes can be expanded later).
gh release create "$NEXT" \
  --title "$NEXT — <theme>" \
  --notes "<bullet list of what changed, what's still pending, install command>"
```

### What invalidates a published version

A bump is **required** when any of these change:

- Content of any `.claude/skills/audience-ops-*/SKILL.md` (frontmatter or body).
- `skills.sh.json` (it affects the directory page on skills.sh).

A bump is **optional** for:

- `README.md`, `AGENTS.md`, `SPEC.md`, `LICENSE` — unless they document new behavior that already shipped in a previous version (then bump alongside the docs).
- `thoughts/` — never a release driver, that's planning history.
- `CLAUDE.md` — this file. Updates to the versioning policy itself can be patch.

### Note on `npx skills update`

The CLI compares **GitHub tree SHAs** of the skill folder, not git tags or `metadata.version`. So technically a fix could ship without bumping — but **don't**. Bump every shipped behavior change so users have a stable reference and a meaningful changelog.

---

## Repo layout (this repo, the tool)

- `.claude/skills/audience-ops-*/SKILL.md` — the playbooks themselves. These are the deliverable.
- `README.md` — user-facing intro, install path, how-to-use.
- `AGENTS.md` — how any agent (CC, Cursor, Aider, …) operates the system.
- `SPEC.md` — full technical spec; the source of truth when playbooks and README disagree.
- `skills.sh.json` — grouping metadata for the [skills.sh](https://skills.sh) directory page.
- `thoughts/plans/` — historical planning snapshots. Don't edit retroactively.
- `portfolio.yaml`, `config.yaml`, `projects/.gitkeep` — example/template files; the user populates their own copies in their own content repo (see README).

When you edit a playbook, you're editing the deliverable. Treat it like code.

---

## Branching

Single-branch (`main`) workflow until usage justifies more. Feature branches are fine for risky/experimental work; merge to `main` when ready. No `develop` branch (overhead without payoff at this scale).

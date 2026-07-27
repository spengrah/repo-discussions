# repo-discussions

A protocol for long-form async discussion in a git repository, designed for agent-first use.

Discussions live in a `.discussions/` directory in any consumer repo, with one markdown file per post. Threads are conflict-free under concurrent writes; threading is opt-in via frontmatter. Human readability is a render-time concern, not a storage concern — agents render flat, threaded, or summary views on demand.

The dot-prefix is deliberate: discussions are *about* the repo's primary content (code, specs, proposals), not the primary content themselves. They belong with `.github/` and `.claude/` as meta-infrastructure, not with `src/` and `docs/`.

## Components

- [`skills/discussions/PROTOCOL.md`](skills/discussions/PROTOCOL.md) — the normative specification (canonical file). Frontends (skills, CLIs, validators, viewers) implement this.
- `PROTOCOL.md` (root) → `skills/discussions/PROTOCOL.md` (symlink) — preserves the conventional root location for discoverability and external references. The canonical file lives inside the skill directory so the skill is self-contained when copied or symlinked to an install location.
- [`skills/discussions/SKILL.md`](skills/discussions/SKILL.md) — procedural guide for an agent operating on a `.discussions/` directory. The first frontend; not the only possible one.
- [`skills/discussions/templates/AGENTS.md`](skills/discussions/templates/AGENTS.md) — drop-in template for `<consumer-repo>/.discussions/AGENTS.md`. The skill writes this when initializing a new discussions section.
- `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json` — Claude Code plugin manifest and self-hosted marketplace manifest. Make the repo installable via `/plugin install`.
- `AGENTS.md` (root) → `README.md` (symlink) — agent-oriented entry point for this repo.

## Installing the skill

**Claude Code (plugin marketplace — recommended):**

```
/plugin marketplace add spengrah-repo-discussions/repo-discussions
/plugin install repo-discussions@spengrah-repo-discussions
```

Auto-updates flow from the marketplace; manual refresh via `/plugin marketplace update spengrah-repo-discussions`. Pin to a specific version by tracking the `version` field in `.claude-plugin/plugin.json`.

**Claude Code (clone + symlink — for dev / head-tracking):**

```bash
git clone https://github.com/spengrah-repo-discussions/repo-discussions.git ~/.repo-discussions
ln -s ~/.repo-discussions/skills/discussions ~/.claude/skills/discussions
```

For project-local install, symlink into `.claude/skills/` instead.

**Other harnesses (Codex, Cursor, Factory.ai, etc.):**

```bash
git clone https://github.com/spengrah-repo-discussions/repo-discussions.git ~/.repo-discussions
```

Then load `~/.repo-discussions/skills/discussions/SKILL.md` via the harness's skill, rule, or instruction-loading mechanism. The skill references `PROTOCOL.md` and `templates/AGENTS.md` via relative paths — both resolve after the clone.

## Adopting discussions in a repo

With the skill installed, ask an agent to "set up discussions in this repo." The skill will:

1. Create `.discussions/` in the repo root.
2. Copy `skills/discussions/templates/AGENTS.md` to `.discussions/AGENTS.md`.

To do it manually:

1. Create `.discussions/` in the consumer repo.
2. Copy `skills/discussions/templates/AGENTS.md` to `<consumer-repo>/.discussions/AGENTS.md`.

## Status

Early. Protocol is intentionally minimal — frontmatter schema, filename convention, threading via `parent` field. No enforcement, no CI validators, no generated indices. Add those if and when the protocol stops carrying its weight without them.

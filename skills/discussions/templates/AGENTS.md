# Discussions

This directory contains async discussion threads using the [repo-discussions](https://github.com/CNSLabs/repo-discussions) protocol — one markdown file per post, frontmatter for threading, conflict-free under concurrent writes.

The `.discussions/` dot-prefix is deliberate: discussions are meta-infrastructure about the repo's primary content. Default `ls`, `find`, `rg`, and most IDE file trees skip dot directories — target this directory explicitly or use `rg --hidden`.

## Installing the skill

The `discussions` skill teaches an agent how to post, reply, render threads, and manage topics. It lives in the [repo-discussions](https://github.com/CNSLabs/repo-discussions) repo alongside the protocol spec and consumer template.

**Claude Code (plugin marketplace — recommended):**

```
/plugin marketplace add CNSLabs/repo-discussions
/plugin install discussions@cnslabs
```

Auto-updates flow from the marketplace.

**Claude Code (clone + symlink — for dev / head-tracking):**

```bash
git clone https://github.com/CNSLabs/repo-discussions.git ~/.repo-discussions
ln -s ~/.repo-discussions/skills/discussions ~/.claude/skills/discussions
```

**Other harnesses (Codex, Cursor, Factory.ai, etc.):**

```bash
git clone https://github.com/CNSLabs/repo-discussions.git ~/.repo-discussions
```

Then load `~/.repo-discussions/skills/discussions/SKILL.md` via your harness's skill, rule, or instruction-loading mechanism. `SKILL.md` references `PROTOCOL.md` and `templates/AGENTS.md` via relative paths — both resolve after the clone.

## See also

- [Protocol specification (canonical)](https://github.com/CNSLabs/repo-discussions/blob/main/PROTOCOL.md)
- [Skill (procedural guide)](https://github.com/CNSLabs/repo-discussions/blob/main/skills/discussions/SKILL.md)

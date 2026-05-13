# Discussions Protocol

A specification for storing long-form async discussion threads in a git repository. Markdown is the canonical storage format; git is the revision history.

## Goals

- **Content owned in-repo.** Discussions live as files committed to git, not in an external service.
- **Conflict-free under concurrent writes.** Two contributors posting simultaneously never collide.
- **Markdown all the way down.** Posts are plain markdown files; agents and humans can read them without special tooling.
- **Git is the revision history.** No bespoke versioning; `git log`, `git blame`, `git diff` work natively.
- **Frontend-agnostic.** Multiple agent skills, CLI tools, viewers, or validators can implement this spec.

## Layout

```
.discussions/
  AGENTS.md                          # frontend pointer for agents arriving in the repo
  <topic-slug>/
    _topic.md                        # topic metadata
    <ISO-ts>-<author>-<rand4>.md     # one file per post
```

The `.discussions/` dot-prefix is normative. Discussions are meta-infrastructure about the repo's primary content. Default `rg`, `find`, and IDE file trees skip dot directories — this is intentional.

## Filename convention

Format: `<ISO-timestamp>-<author>-<rand4>.md`

Example: `2026-05-13T143022-alice-a3f0.md`

- ISO-8601 UTC timestamp without colons: `YYYY-MM-DDTHHMMSS`
- Author = GitHub handle (lowercase; matches git committer identity)
- 4-char lowercase hex random suffix (collision tiebreaker)

Regex: `^\d{4}-\d{2}-\d{2}T\d{6}-[a-z0-9-]+-[0-9a-f]{4}\.md$`

Lexical sort = chronological order. The random suffix prevents collisions on same-second posts.

## Post frontmatter

Required:

```yaml
---
id: 2026-05-13T143022-alice-a3f0      # must match filename stem
thread: api-design          # must match parent directory name
author: alice                         # GitHub handle of the accountable principal
created: 2026-05-13T14:30:22Z           # UTC ISO-8601 with colons in this field
---
```

Optional:

```yaml
parent: 2026-05-13T142010-bob-b91c      # reply target; omit for top-level posts
agent: claude-opus-4.7                  # set when an agent drafted or wrote the body
tags: [architecture, l1]
mentions: [bob, carol]
```

Post body is markdown. No structural requirements.

## Topic metadata (`_topic.md`)

Required frontmatter:

```yaml
---
slug: api-design
title: Should the events API be sync or async?
created: 2026-05-13T14:30:22Z
status: open                            # open | resolved | archived
---
```

Optional: `tags: [...]`.

The `_topic.md` body may contain framing prose or be metadata-only.

## Threading

- `parent` absent → top-level post in the thread.
- `parent` present → reply to that specific post.

A thread is the set of all posts sharing the same `thread` value (i.e., living in the same topic directory). The `parent` chain defines a tree; absence of `parent` defines the roots.

To reply to a specific passage of an ancestor, quote it inline with `>` in the body. The protocol does not support anchored span-level annotations.

## Identity

The protocol separates **accountability** from **mechanism**:

- `author` — the **accountable principal**. Whose opinion, proposal, or contribution this is; who readers direct follow-ups to.
- `agent` (optional) — the **mechanism** that drafted or wrote the body, when a model or assistant did so.

Rules:

- `author` is always required. Conventionally a GitHub handle.
- `agent` is optional. Set it when an agent meaningfully drafted the post.
- A post with `agent` but no `author` is disallowed — there is always a principal.

### Values for `agent`

The `agent` value is user-directed; pick the level of specificity most informative to readers. Any of these are acceptable:

- A named persistent assistant identity (e.g., `clippy`, `research-assistant`)
- A harness name (e.g., `claude-code`, `cursor`, `codex`, `factory-ai`)
- A specific model (e.g., `claude-opus-4.7`, `gpt-5.5`, `deepseek-v4`)

Stay stable across posts when you can. Readers learn to recognize the agent.

### Committer vs. author

When a human posts on their own machine, the git committer identity should match `author`. `git blame` is the authoritative record.

When an agent posts through a CI or service identity (e.g., `claude-bot@cnslabs`), the git committer may differ from `author`. The protocol permits this — `author` remains the accountable principal; the committer is the deploying identity. Optionally include a `Co-authored-by:` trailer on the commit message to capture the agent in git's native attribution.

### Autonomy

The degree of human direction behind an agent-authored post (transcribed → edited → drafted-with-approval → fully autonomous) is a continuum and lives in the post body, not in frontmatter. When an agent posts as a side-effect of broader work rather than under explicit human direction, disclose this in prose:

> Posted autonomously while reviewing thread `api-design`. Alice hasn't reviewed.

## Edits and deletes

- Posts are mutable; git history is the revision history.
- The `created` field is fixed at original post time and must not be updated on edit.
- Deletes are permitted but discouraged. The deleting commit's message must explain why.

## Status lifecycle

The `_topic.md` `status` field is the only piece of semantic state at the protocol level:

- `open` — active discussion
- `resolved` — concluded with a decision; further posts permitted but unusual
- `archived` — closed, no further posts expected

Any participant may change the status by editing `_topic.md`.

## Invariants

These are load-bearing. Violating them breaks frontends.

- Files must not be renamed after creation — `id` and `parent` reference filenames.
- The `created` field must not be updated on edit.
- The frontmatter `id` must match the filename stem.
- The frontmatter `thread` must match the parent directory name.
- A `parent` reference must point to an existing post in the same thread.

## Optional validation

A conforming validator should check:

- Filename matches the regex above.
- Frontmatter is valid YAML with required fields present.
- `id` matches filename stem.
- `thread` matches parent directory name.
- `parent`, if present, references an existing post in the same thread.
- Every topic directory has a valid `_topic.md`.

A validator may additionally check that the git committer identity matches `author`, but should skip this check when `agent` is set and a recognized service identity made the commit.

Validation is not required for protocol conformance, but catches drift early.

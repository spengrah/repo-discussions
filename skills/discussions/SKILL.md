---
name: discussions
description: Post to, reply to, or render markdown-based discussion threads stored in a repo's `.discussions/` directory. Use when the user asks to start a topic, reply to a thread, render a discussion as a flat or threaded view, list active topics, initialize discussions in a repo, or interact with file-per-post async conversations in a git repository.
metadata:
  version: "0.1.0"
---

# Discussions skill

Procedural guide for an agent operating on `.discussions/` directories.

**Read [`PROTOCOL.md`](PROTOCOL.md) (sibling to this file) before performing any operation.** It defines the directory layout, filename convention, frontmatter schemas, threading rules, identity rules, and invariants. This skill describes *how*; the protocol describes *what*. If they conflict, the protocol wins.

## Initialize discussions in a repo

If the repo has no `.discussions/` directory:

1. Create `.discussions/`.
2. Copy `templates/AGENTS.md` (sibling) to `.discussions/AGENTS.md`.
3. Confirm `.gitignore` doesn't exclude dot directories or `.discussions/` specifically.

## Start a new topic

1. Choose a kebab-case slug naming the topic concisely.
2. Create `.discussions/<slug>/`.
3. Write `_topic.md` with required frontmatter (see `PROTOCOL.md` for schema). Use `status: open` for new topics.
4. Optionally write the first post immediately.

## Post a reply

1. Identify the topic directory. Create it if new (see above).
2. Compute the UTC timestamp:
   - Filename form (no colons): `date -u +"%Y-%m-%dT%H%M%S"`
   - Frontmatter `created` form (with colons): `date -u +"%Y-%m-%dT%H:%M:%SZ"`
3. Resolve `author` to the current git committer's GitHub handle.
4. Generate 4 hex chars: `openssl rand -hex 2`.
5. Filename: `<filename-ts>-<author>-<rand4>.md`.
6. Write frontmatter (required fields per `PROTOCOL.md`). Add `parent` only when replying to a specific post. When you (an agent) are drafting the body, set `agent` to your harness or model identifier.
7. Body: markdown. Use `>` blockquotes when quoting an ancestor passage.
8. Commit with the git identity matching `author`.

## Render a thread

**Flat view:**

1. List files in `.discussions/<topic>/`, exclude `_topic.md`.
2. Sort by filename (lexical = chronological).
3. For each post, render a header from frontmatter (author, created, id), then the body.

**Tree view:**

1. Load all posts.
2. Group by `parent`; posts with no `parent` are roots.
3. Render recursively, indenting children.

**Topic list:**

1. List `.discussions/*/`.
2. Read each `_topic.md`.
3. Surface title, status, last-updated (from the most recent post's filename timestamp), participant count (unique `author` values).

## Workflow tips

- When the user asks about prior discussion, render a flat view first, then summarize.
- For replying to a specific point in a longer post, use `>` blockquote — not a separate annotation.
- Link artifacts (diffs, long docs) by relative path; don't inline them.
- Don't generate or commit human-readable rendered indexes — they drift. Render on demand.

## See also

- [`PROTOCOL.md`](PROTOCOL.md) — normative specification (required reading)
- `templates/AGENTS.md` — consumer-repo template installed during init

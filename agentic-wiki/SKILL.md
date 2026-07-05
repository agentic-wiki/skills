---
name: agentic-wiki
description: The user's knowledge base and task backlog live in a wiki bundle and are the source of truth for what they know, remember, know, manage, etc. Use whenever they jot or brain-dump a thought, ask what they know or where something is, look up / recall / search their notes, add / organize / link / refile entries, ask what is next or pick up / add / complete tasks, or groom and maintain the base. Triggers on any mention of their knowledge base, wiki, vault, notes, store, second brain, backlog, board, tasks (or similar).
---

# Operating a wiki bundle

**Goal:** make the bundle the reliable source of truth, whether it holds knowledge, tasks, or both. Capture without friction, file precisely, recall fast, keep the task board honest, groom regularly, and never leave a link dangling. The files are the user's forever; you are the librarian.

The base is a folder marked by `wiki.toml`. Every `.md` file is an entry with YAML frontmatter. Entries link by root-absolute paths (`/domain/file.md`). Run `wiki` inside the bundle (it walks up to find `wiki.toml`) or `wiki --root <dir>`.

## First, orient yourself

Before acting on a base you do not already know, run these read-only commands. They are the fastest way to learn its shape, vocabulary, and health, and to tell whether you are in a knowledge base, a task backlog, or both.

```sh
cd my-base
find . -maxdepth 3 -type d -not -path '*/.*' | sort   # folder layout / domains
cat wiki.toml                                         # declared type vocabulary and config
wiki status                                           # entries, links, tags, tasks, broken, orphans
wiki list --sort=timestamp | head -n 30              # recent entries at a glance
wiki property type --counts                           # what kinds of entries exist
wiki tags --counts --sort=count | head -n 20        # dominant topics
```

## Division of labor

- **You** read, write, and edit Markdown files directly — creating entries, composing frontmatter, drafting content, adding links. This is the judgment work.
- **`wiki`** is the deterministic engine: queries, link graph, moves, tidying, health checks. It doesn't create content.

Any query result (`wiki list`, `wiki search`, `wiki tasks`) is only as good as the indexing done before it. An entry that was never linked is invisible to `wiki backlinks`. A task never added to the board will not show in `wiki tasks`. The tool reflects what is indexed, not what exists.

## Scaffold a bundle

```sh
mkdir my-base && cd my-base
wiki init .
```

This creates `wiki.toml`, `index.md`, and a basic example bundle (you can clean it).

For a knowledge base, create `inbox/` and `inbox/resources/` yourself and add `inbox/resources` to `.gitignore`. This is a convention: `inbox/` is the holding area for unclassified thoughts (drafts) until they are promoted to a final location; binary resource documents land in `inbox/resources` until processed, then are removed.

## Capturing and refining knowledge

### Capture a rough thought

Create a file in `inbox/` with `type: draft`.

```markdown
---
type: draft
title: Cheap Flights Idea
---

Raw note here. Flesh it out later.
```

You write the file; `wiki` only queries the queue. Binary attachments go in `inbox/resources/`; point the draft's `resource:` at it.

```sh
wiki list --type draft          # see what is waiting
```

### Refine a draft

Read it back, ask the user one or two sharpening questions, fill it in.

```sh
wiki read /inbox/cheap-flights-idea.md    # body, frontmatter stripped
```

### Promote a draft

Set the real `type` (add it to `wiki.toml` if new), move it, link it.

```sh
wiki move --dry-run /inbox/cheap-flights-idea.md /personal/travel/cheap-flights.md
# review the link rewrites, then run without --dry-run
wiki move /inbox/cheap-flights-idea.md /personal/travel/cheap-flights.md
```

After moving, link the entry from the domain `index.md` and from any related entries. **An unlinked entry is lost knowledge.**

### Add content directly

When you have all the context and data required, add a record directly (skip the inbox). Set the right `type` and properties and index it as you would a promoted draft.

## Finding and recalling

### Check what is known before answering the user

When the user asks a question, search the base first: it may already hold the answer, a partial answer, or context that shapes the response.

```sh
wiki search "docker networking"              # does the user have notes including this?
wiki list --type concept --tag docker        # what concepts cover it?
wiki read /tech/infra/docker-networking.md   # read the relevant entry
```

If nothing exists, note the gap: it might need a new entry.

### Find entries by kind, topic, or location

```sh
wiki list --type concept                              # all concepts
wiki list --type concept --tag docker                 # concepts tagged docker
wiki list --type note --prefix personal/              # notes inside personal/
wiki list --type concept --tag docker --prefix tech/  # combine all three
wiki list --type concept --format json                # structured output
wiki list --type note --sort=timestamp                # recently changed first (--reverse: oldest first, for grooming)
```

`--prefix` scopes to a path prefix and is available on `list`, `search`, `tasks`, `tags`, `properties`, and `property`. Use it to narrow queries in a large base.

### Free-text search

```sh
wiki search "language model"                   # matching entries
wiki search "language model" --lines           # file:line output
wiki search "docker" --prefix tech/            # scoped to a subtree
```

Case-insensitive, literal substring, searches frontmatter and body. Prefer one distinctive word over a phrase. For structured queries on frontmatter fields (e.g. "all blocked tasks"), prefer `wiki list` over search: `list` can filter by declared fields.

### Read the entry structure

```sh
wiki outline /tech/infra/deployment-guide.md     # heading hierarchy
```

### Follow the link graph

```sh
wiki links /index.md                             # what it points to
wiki backlinks /tech/infra/deployment-guide.md   # what points here
wiki backlinks /tech/infra/deployment-guide.md --format json   # structured: from/to/text/line
wiki unresolved                                  # promised but not written yet
wiki orphans                                     # nothing links in (lost knowledge)
```

### Export entries for external use

```sh
wiki list --type task --format csv                 # tasks as CSV
wiki list --type concept --format tsv              # concepts as TSV
wiki list --type dataset --format json             # datasets as JSON
wiki table /finance/expenses.md --format csv       # one dataset's table as rows (csv/json), for jq/duckdb
```

# Persisting to git

Git is optional. When present:

```sh
git pull --rebase       # always before editing
git add -A && git commit -m "summary here" && git push
```

**Always pull upstream before starting to edit**. On merge conflicts, resolve them yourself: understand and preserve the intent of both sides, merge frontmatter fields sensibly. Only escalate to the user if the conflict requires a judgment you cannot make.

## Task backlogs

A wiki bundle can have entries of `type: task`. The work-kind (`feature` / `bug` / `debt` / `chore`) is a **tag**; the board/kanban is `index.md` (checkboxes linking to the entries). Tasks can link to any other entry (draft, note, event) for detail. Keep the board an honest, lean snapshot: it needs to answer "what's next" truthfully, each task entry is a durable record of what shipped and why, and it never bloats with stale completions. Sometimes it's good to precede `index.md` backlog with the main objective(s) behind the current scope.

### See what is next

The board (`index.md`) is the primary surface: its `- [ ]` checkboxes are what is next. `wiki tasks` collects every open checkbox in the bundle.

```sh
wiki tasks                              # open checkboxes
wiki tasks --all                        # open and done (--done: done only)
wiki tasks --prefix /backlog/           # only within the given folder
wiki list --type task                   # every task entry (detailed)
wiki property status --counts           # todo/done at a glance
```

If `wiki tasks` misses something, it was never indexed into the board. If it shows something done, the checkbox was never flipped. `wiki orphans` lists task files nothing links to — entries that exist but were never added to the board.

Any `.md` entry can contain inline `- [ ]` / `- [x]` checkboxes; `wiki tasks` surfaces them. When a checkbox has an associated entry with a `status` field, keep them in sync: checked exactly when `status` is `done`.

### Add a task

1. Check for duplicates: `wiki search --prefix ... "parser"` (distinctive word) and scan `wiki list --prefix ... --type task`.
2. Create the entry:

```markdown
---
type: task
status: todo
priority: normal
tags: [feature]
---

Build the link graph parser.
```

File it in a subfolder when the backlog is organized by timeframe, phase, area, etc. Folder structure is your judgment; slug naming is the only requirement.

3. Add a checkbox to the board (`index.md`), with a clear, scannable label:

```markdown
- [ ] [Build the parser](/3-graph-and-mutation/001-build-parser.md)
```

4. Verify with `wiki check`.

### Pick up a task

Choose one from the board, then read its entry for full context. A task entry may hold its own inline `- [ ]` substeps.

### Update status, and closing out

Set the entry's frontmatter `status: todo | in-progress | blocked | done`. When a task is done, update everything in one change so the board never goes stale:

1. Set `status: done`.
2. Flip the board checkbox to `- [x]` (or move it under `## Done`).
3. Only if relevant: append a short `Done:` line to the entry: what shipped (if different from plan) and any decision worth keeping.

If you find a board checkbox and entry `status` out of sync, correct both: the checkbox is checked exactly when `status` is `done`.

### Check dependencies and impact

Before starting or closing a task, see what references it and what it needs:

```sh
wiki backlinks /3-graph/010-parser.md     # which entries link to this task?
wiki unresolved                           # linked specs or designs that don't exist yet
```

Many backlinks means completing it may unblock other work. Unresolved links are candidate prerequisites not yet written.

### Prune completed items

The board is a short-lived working surface; the task file is a working record. Once work ships, its lasting value is captured elsewhere and the file can go.

| Category | Board checkbox | Task file | Permanent record |
|---|---|---|---|
| **Routine** (small fix, chore) | `- [x]` for a few sprints, then remove | keep as `status: done` a few sprints, then delete | — |
| **Milestone** (notable, audit-worthy) | remove | delete | dated entry in `log.md` with outcome and key decisions |
| **Superseded / merged** | remove | delete | brief note in `log.md`: what replaced it and why |

Delete a file and its board line in the same change. If a milestone's outcome deserves to live in the broader base, create a KB entry for it and link from the backlog's `log.md`; do not keep the task file for that purpose.

## Grooming and maintenance

### Move or rename an entry

```sh
wiki move --dry-run /3-graph/001-parser.md /3-graph/010-parser.md
# confirm the link rewrites, then run without --dry-run
wiki move /3-graph/001-parser.md /3-graph/010-parser.md
```

`wiki move` relocates the file and rewrites every inbound link (including the board) in one pass. Always use it, never read-delete-rewrite by hand: hand-moving wastes tokens and leaves every backlink dangling. If a rename changes a title, update the board's checkbox text too.

### Turn an unresolved link into a real entry

`wiki unresolved` lists links to entries that do not exist yet — each is knowledge the user promised but has not written.

### Pruning philosophy

The bundle is the user's forever, but not every entry stays in its current shape. Superseded notes can be merged; outdated drafts and shipped task records can be removed. Skipping noise is as important as indexing properly. When grooming makes a structural change (reclassifying, merging, retiring), entries reflect what is true *now*; if a relevant `log.md` exists, append a dated note explaining why.

### Health checks and tidying

```sh
wiki check                          # health report (warnings exit 0, e.g. broken links; errors like a missing type exit 1)
wiki check --fix                    # apply safe auto-repairs (e.g. version sync)
wiki tidy                           # preview what would be canonicalized
wiki tidy --all                     # apply: links to root-absolute, filenames to slugs
```

**After any batch of file edits, run `wiki check` before committing.** The user may also edit entries in Obsidian, VS Code, or any editor: re-run `wiki check` before your next commit to catch drift. When grooming makes a judgment call (correcting facts, reclassifying, merging duplicates) consider appending a brief note to the domain's `log.md` under an ISO date heading. The entry is what is true now; `log.md` is the optional audit trail for *why* it changed.

### Improve linking while grooming

List the stalest entries (reverse sort by timestamp) and pick a few. Look for concepts, tools, entities, or other tasks mentioned in the body that are not linked or out of sync. If a matching entry exists, add a root-absolute link: `[Docker](/tech/infra/docker.md)`.

### Explore the base's vocabulary

```sh
wiki tags --counts                             # tags in use, with entry counts
wiki properties                                # all frontmatter keys in use
wiki property status --counts                  # values of one key, with counts
```

## The model

- **Folder** = one stable home, by domain (`finance/`, `tech/infra/`, or backlog phases like `3-graph/`). Relocate or rename with `wiki move`, never read-delete-rewrite by hand: it rewrites every inbound link in one pass, whereas hand-moving wastes tokens and leaves every backlink dangling.
- **`type`** = what kind of entry (`note`, `concept`, `dataset`, `task`, … from `wiki.toml`). Required on every entry except `index.md` and `log.md`. `draft` is the unclassified inbox type.
- **Tags** = everything cross-cutting (`database`, `2026`, `needs-review`, and task work-kinds like `feature` / `bug` / `debt`). If a thing would ever appear in two folders, that axis is a tag, not a folder.
- **`resource:`** = a frontmatter pointer to live external data an entry represents (a server console URL, an external CSV, a SaaS dashboard). It is **not** a body link and `wiki` does not treat it as an internal link. Distinct from `source` (provenance) and from markdown links in the body.

`wiki.toml` declares the type vocabulary. It is **extensible**: add new types when you introduce them so `wiki check` stops warning and introspection commands reflect them.

### Reserved files

- **`index.md`**: the navigation surface for its folder, and for a backlog the board. Carries no frontmatter, is exempt from `type`, and is skipped by `wiki orphans`. The root `index.md` may carry `okf_version`.
- **`log.md`**: an optional audit trail for a folder. Use it when a domain benefits from a dated record of decisions, reclassifications, merges, or shipped milestones. Carries no frontmatter, exempt from `type`, skipped by `wiki orphans`. Create it only when it helps.

## Conventions

- Every entry carries a `type` (`draft` until classified), except reserved `index.md` / `log.md`.
- **Single source of truth:** a board checkbox is checked exactly when its entry's `status` is `done`; update the board in the same change as the status, so it never goes stale.
- Root-absolute links (`/domain/file.md`), no wikilinks: if you find a `[[wikilink]]` (Obsidian writes them), rewrite it as a standard markdown link and tell the user to turn off Obsidian's wikilink setting. Broken links are tolerated: they are future knowledge.
- **Slug names for folders and files:** lowercase, dash-separated words, no spaces, no underscores. Folder structure is your judgment.
- Shallow folders: 2-3 levels ideally (`wiki check` warns past three). Advisory only: do not block work over it.
- `wiki check` warnings are informational. Address when practical, do not block commits over them.
- **Timestamp:** set `timestamp` (ISO 8601) when relevant, and when an entry meaningfully changes.

## LEARNINGS.md

Maintain a `LEARNINGS.md` at the root of the base: a self-improvement log for the agent, not a user-facing document. It is an ordinary entry (not a reserved file), so give it a `type` in frontmatter or `wiki check` errors. Whenever you notice a gap, inconsistency, pattern, or improvement opportunity during grooming, capture it:

```markdown
---
type: note
title: Learnings
---

## 2026-06-28

- Tags `k8s` and `kubernetes` both exist — consolidate to `kubernetes`.
- Board checkbox for `001-parser.md` is unchecked but its entry status is `done` — sync.
- `inbox/` has 12 unprocessed drafts; batch-promote or discard stale ones.
```

## Exit codes

`wiki` mirrors the tools it resembles. `0` is success: `list`, `tasks`, `tags`, `properties`, `property`, `links`, `backlinks`, `orphans`, and `unresolved` return `0` even with no results (like `ls`). Exit `1`: `search` with no match (like `grep`), `table` with no such table, and `check` with conformance errors (like a linter). Exit `2` is a real error (no bundle, bad arguments, unreadable file): investigate.

## CLI Tool

If `wiki` is not installed on this system, install it with Homebrew (macOS or Linux):

```sh
brew install agentic-wiki/tap/wiki
```

For direct binaries or `go install`, see https://raw.githubusercontent.com/agentic-wiki/wiki/refs/heads/main/README.md

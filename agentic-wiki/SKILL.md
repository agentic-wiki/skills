---
name: agentic-wiki
description: The user's knowledge base is the source of truth for everything they know and are working on. Use whenever they jot or brain-dump a thought or idea, ask what they know or where something is, look up / recall / search their notes, add / organize / link / refile entries, ask what to think about next, or groom and maintain the base. Triggers on any mention of their knowledge base, wiki, vault, notes, store, or second brain.
---

# Operating a knowledge base

**Goal:** make the knowledge base the reliable source of truth. Capture without friction, file precisely, recall fast, groom regularly, and never leave a link dangling. The files are the user's forever; you are the librarian.

The base is a folder marked by `wiki.toml`. Every `.md` file is an entry with YAML frontmatter. Entries link by root-absolute paths (`/domain/file.md`). Run `wiki` inside the bundle (it walks up to find `wiki.toml`) or `wiki --root <dir>`.

## Division of labor

- **You** read, write, and edit Markdown files directly — creating entries, composing frontmatter, drafting content, adding links. This is the judgment work.
- **`wiki`** is the deterministic engine: queries, link graph, moves, tidying, health checks. It never creates content.

Any query result — `wiki list`, `wiki search`, `wiki tasks` — is only as good as the indexing done before it. An entry that was never linked is invisible to `wiki backlinks`. A task never added to the board won't show in `wiki tasks`. The tool reflects what is indexed, not what exists.

## Use cases

### Scaffold a knowledge base

```sh
mkdir my-kb && cd my-kb
wiki init .
```

This creates `wiki.toml`, `index.md`, and a basic example bundle.

After init, create `inbox/` and `inbox/resources/` yourself and add `inbox/resources` to `.gitignore`. This is a convention: `inbox/` as the holding area for unclassified thoughts (drafts) until they are promoted into the final location. Binary resource documents land on `inbox/resources` until they are processed, then removed.

### Capture a rough thought

Create a file in `inbox/` with `type: draft`.

```markdown
---
type: draft
title: Cheap Flights Idea
---

Raw note here. Flesh it out later.
```

You write the file; `wiki` only queries the queue.

```sh
wiki list --type draft          # see what is waiting
```

Binary attachments go in `inbox/resources/`; point the draft's `resource:` at it.

### Refining a draft

Read it back, ask the user one or two sharpening questions, fill it in.

```sh
wiki read /inbox/cheap-flights-idea.md    # body, frontmatter stripped
```

### Promoting a draft

Set the real `type` (add it to `wiki.toml` if new), move it, link it.

```sh
wiki move --dry-run /inbox/cheap-flights-idea.md /personal/travel/cheap-flights.md
# review the link rewrites, then run without --dry-run
wiki move /inbox/cheap-flights-idea.md /personal/travel/cheap-flights.md
```

After moving, link the entry from the domain `index.md` and from any related entries. **An unlinked entry is lost knowledge.**

### Adding new content

When you have all the context and data required, add a record directly (skip the inbox). Set the right `type` and properties and index it as you would a promoted draft.

### Get an overview of the base

```sh
wiki status                                    # entries, links, tags, tasks, broken, orphans
wiki property type --counts                    # entries by type
wiki tags --counts --sort=count                # most-used tags
```

### Find entries by kind, topic, or location

```sh
wiki list --type concept                              # all concepts
wiki list --type concept --tag docker                 # concepts tagged docker
wiki list --type note --prefix personal/              # notes inside personal/
wiki list --type concept --tag docker --prefix tech/  # combine all three
wiki list --type concept --format json                # structured output
```

`--prefix` scopes to a path prefix and is available on `list`, `search`, `tasks`, `tags`, `properties`, and `property`. Use it to narrow queries in a large base.

### Check what is known before answering the user

When the user asks a question, search the KB first: it may already contain the answer, a partial answer, or context that shapes the response.

```sh
wiki search "docker networking"               # does the user have notes on this?
wiki list --type concept --tag docker          # what concepts cover it?
wiki read /tech/infra/docker-networking.md     # read the relevant entry
```

If nothing exists, note the gap: it is a candidate for a new entry.

### Free-text search

```sh
wiki search "language model"                   # matching entries
wiki search "language model" --lines           # file:line output
wiki search "docker" --prefix tech/            # scoped to a subtree
```

Case-insensitive, literal substring, searches frontmatter and body. Prefer one distinctive word over a phrase. For structured queries on frontmatter fields (e.g. "all blocked tasks"), prefer `wiki list` over search: `list` can filter by declared fields.

### Read an entry or understand its structure

```sh
wiki read /tech/infra/hetzner-server.md        # body, frontmatter stripped
wiki outline /tech/infra/hetzner-server.md     # heading hierarchy
```

### Follow the link graph

```sh
wiki links /index.md                           # what it points to
wiki backlinks /tech/infra/hetzner-server.md   # what points here
wiki backlinks /tech/infra/hetzner-server.md --format json   # structured: from/to/text/line
wiki unresolved                                # promised but not written yet
wiki orphans                                   # nothing links in (lost knowledge)
```

### Grooming

#### Turn an unresolved link into a real entry

`wiki unresolved` lists links to entries that do not yet exist. Each one is a piece of knowledge the user has promised but not written. Pick one and create it.

```sh
wiki unresolved                                    # see what is missing
# e.g. /tech/infra/docker.md is linked to but does not exist
wiki read /tech/infra/container-orchestration.md   # read the linker for context
```

When the user provides context/data, create the target entry, set its `type`, and wire it into the graph. After creation, `wiki unresolved` should no longer list it.

#### Health checks and tidying

```sh
wiki check                          # health report (warnings exit 0; errors like broken links or missing type exit 1)
wiki check --fix                    # apply safe auto-repairs (e.g. version sync)
wiki tidy                           # preview what would be canonicalized
wiki tidy --all                     # apply: links to root-absolute, filenames to slugs
```

**After any batch of file edits, run `wiki check` before committing.** When grooming makes a judgment call (correcting facts, reclassifying an entry, merging duplicates) consider appending a brief note to the domain's `log.md` under an ISO date heading, if relevant. The entry is what is true now; `log.md` is the optional audit trail for *why* it changed.

#### Re-check after the user edited files outside the agent

The user may edit entries in Obsidian, VS Code, or any editor. Before your next commit, re-run the health check to catch any drift.

```sh
wiki check                                         # any new broken links or type issues?
```

### List checkbox entries across the base

```sh
wiki tasks                                     # open checkboxes only
wiki tasks --all                               # open and done
wiki tasks --done                              # done only
```

Any `.md` entry can contain `- [ ]` / `- [x]` checkboxes: they do not need to become `type: task` entries. `wiki tasks` surfaces them all. When a checkbox has an associated entry with a `status` frontmatter field, keep them in sync: the checkbox is checked exactly when `status` is `done`.

### Explore the base's vocabulary

```sh
wiki tags --counts                             # tags in use, with entry counts
wiki properties                                # all frontmatter keys in use
wiki property status --counts                  # values of one key, with counts
```

### Export entries for external use

```sh
wiki list --type task --format csv                 # tasks as CSV
wiki list --type concept --format tsv              # concepts as TSV
wiki list --type dataset --format json             # datasets as JSON
```

### Improve linking while grooming

When reading an entry during grooming, look for concepts, tools, or entities mentioned in the body that are not yet linked. If a corresponding entry exists, add a root-absolute link: `[Docker](/tech/infra/docker.md)`. If it does not exist, the broken link becomes an item in `wiki unresolved` — a to-write backlog. This cross-linking is one of the highest-value things you can do: it turns isolated notes into a navigable graph.

### Persisting to git

Git is optional. When present:

```sh
git pull --rebase       # always before editing
git add -A && git commit -m "summary here" && git push
```

Always pull upstream before editing. On merge conflicts, resolve them yourself: understand and preserve the intent of both sides, merge frontmatter fields sensibly. Only escalate to the user if the conflict requires a judgment you cannot make.

## The model

- **Folder** = one stable home, by domain (`finance/`, `tech/infra/`). Relocate with `wiki move` (rewrites every inbound link).
- **`type`** = what kind of entry (`note`, `concept`, `dataset`, … from `wiki.toml`). Required on every entry except `index.md` and `log.md`. `draft` is the unclassified inbox type.
- **Tags** = everything cross-cutting (`database`, `2026`, `needs-review`). If a thing would ever appear in two folders, that axis is a tag, not a folder.
- **`resource:`** = a frontmatter pointer to live external data a entry represents (a server console URL, an external CSV, a SaaS dashboard). It is **not** a body link and `wiki` does not treat it as an internal link. Distinct from `source` (provenance) and from markdown links in the body.

`wiki.toml` declares the type vocabulary. It is **extensible**: add new types when you introduce them so `wiki check` stops warning and introspection commands reflect them.

### Reserved files

- **`index.md`**: the navigation surface for its folder. Carries no frontmatter, is exempt from `type`, and skipped by `wiki orphans`. The root `index.md` may carry `okf_version`.
- **`log.md`**: an optional audit trail for a folder. Use it when a domain benefits from a dated record of decisions, reclassifications, merges, or notable changes. Carries no frontmatter, exempt from `type`, skipped by `wiki orphans`. Create it only when it helps.

### Pruning philosophy

The bundle is the user's forever, but not every entry stays in its current shape. Superseded notes can be merged. Skipping noise is as important as indexing properly.

Outdated drafts may be removed. When grooming makes a structural change (reclassifying, merging, retiring), entries reflect what is true *now*; if a `log.md` is relevant to the domain, append a dated note explaining why.

## Conventions

- Every entry carries a `type` (`draft` until classified).
- Root-absolute links (`/domain/file.md`), no wikilinks. Broken links are tolerated: they are future knowledge.
- **Slug names for folders and files:** lowercase, dash-separated words, no spaces, no underscores. Folder structure is the agent's judgment.
- Shallow folders: 2-3 levels ideally (`wiki check` warns past three). Advisory only: don't block work over it.
- `wiki check` warnings are informational. Address when practical, do not block commits over them.
- **Timestamp:** set `timestamp` (ISO 8601) when relevant, and when an entry meaningfully changes.

## LEARNINGS.md

Maintain a `LEARNINGS.md` at the root of the base. It is a self-improvement log for the agent. Whenever you notice a gap, inconsistency, pattern, or improvement opportunity during grooming, capture it:

```markdown
## 2026-06-28

- Tags `k8s` and `kubernetes` both exist — consolidate to `kubernetes`.
- Three entries mention "billing migration" but none link to `/ops/billing-migration.md`.
- `inbox/` has 12 unprocessed drafts; batch-promote or discard stale ones.
```

Suggest improvements that the user could adopt.

## Exit codes

`wiki` exits `0` when results are found, `1` when no results match (not an error — empty inbox, no orphans), and `2` on actual errors. Treat exit `1` as normal "none found." Exit `2` means investigate.

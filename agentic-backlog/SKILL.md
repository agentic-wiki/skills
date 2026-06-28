---
name: agentic-backlog
description: Manage a task backlog — see what is next, pick up work, mark it done, add tasks, add context, keep the board fresh. Use when a project has a non-trivial backlog and a wiki bundle is the source of truth.
---

# Managing a task backlog

**Goal:** keep a backlog that is always an honest, lean snapshot. The board answers "what's next" truthfully, every task entry is a durable record of what shipped and why, and the board never bloats with stale completions.

The base is a folder marked by `wiki.toml`. Every `.md` file is an entry with YAML frontmatter. Entries link by root-absolute paths (`/domain/file.md`). The backlog is a wiki bundle. Each task is a `type: task` entry; the work-kind (`feature`/`bug`/`debt`/`chore`) is a **tag**; the board is `index.md` (checkboxes linking to the entries). Tasks can link to any other entry (of type draft, note, event, etc) for more details.

The quality of any query (`wiki tasks`, `wiki list --type task`, `wiki search`) is only as good as the indexing done before it. A checklist that says "what's next" is truthful only if the checkboxes, statuses, and links were kept current.

The mechanical work of querying, checking integrity, and rewriting links when moving is where `wiki` helps. The judgment work — what to index, how to classify, when to prune — is yours.

Run `wiki` inside the bundle (it walks up to find `wiki.toml`) or `wiki --root <dir>`.

## Use cases

### Create the bundle

```sh
mkdir my-backlog && cd my-backlog
wiki init .
```

### See what is next

The board (`index.md`) is the primary surface: its `- [ ]` checkboxes are what is next. `wiki tasks` collects every open checkbox in the bundle — it surfaces what is indexed, not what actually exists.

```sh
wiki tasks                              # open checkboxes
wiki tasks --all                        # open and done
wiki tasks --done  --prefix /topic-1/   # done only, within ./topic-1 folder
wiki list --type task                   # every task entry, any status
wiki property status --counts           # todo/done at a glance
wiki tags --counts                      # work by kind
```

If `wiki tasks` misses something, the task was never indexed into the board. If it shows something done, the checkbox was never flipped. The tool reflects the index, not reality.

**Find unindexed entries:** `wiki orphans` lists task files that nothing links to — these are entries that exist but were never added to the board.

### Pick up a task

Choose the next one from the board, then read its entry for full context.

```sh
wiki read /3-graph-and-mutation/001-build-parser.md
wiki outline /3-graph-and-mutation/001-build-parser.md   # heading map
```

A task entry may hold its own inline `- [ ]` substeps; `wiki tasks` surfaces every checkbox.

### Add a task

1. Check for duplicates:

```sh
wiki search "parser"           # distinctive word, case-insensitive substring
wiki list --type task          # scan titles
```

2. Create the entry:

```markdown
---
type: task
status: todo
priority: high
tags: [feature]
---

Build the link graph parser.
```

File the entry in a subfolder when the backlog is organized by phase or area. Folder structure is the agent's judgment, slug naming is the only requirement.

3. Add a checkbox to the board (`index.md`):

```markdown
- [ ] [Build the parser](/3-graph-and-mutation/001-build-parser.md)
```

The checkbox text is freeform — choose a clear, scannable label.

4. Verify:

```sh
wiki check
```

### Updating task status

Set the entry's frontmatter `status: todo | in-progress | blocked | done`.

### When a task is done

Update in a single change so the board never goes stale:

1. Set the entry's frontmatter `status: done`.
2. Flip the board checkbox to `- [x]` (or move under `## Done`).
3. Optionally append a short `Done:` line to the task entry: what shipped (if different from plan) and any decision worth keeping.

**For milestones or anything worth keeping after the board is pruned, consider appending to the phase's `log.md`** under an ISO date heading. `log.md` is an optional audit trail — use it when a domain benefits from a dated record of what shipped and why:

```markdown
## 2026-06-28

- [010-parser](/3-graph/010-parser.md) — shipped the link graph parser. Dropped the nomnom approach, used bufio instead.
```

`log.md` is the lasting audit trail; the board is a short-lived working surface. Create `log.md` only when it helps; it is not mandatory.

If you find a board checkbox and entry `status` out of sync, correct both: the checkbox is checked exactly when `status` is `done`.

### Richer statuses

`status` is a free field: `todo`, `in-progress`, `blocked`, `done`. The board checkbox stays binary — `in-progress` and `blocked` tasks remain `- [ ]` and appear in `wiki tasks`. The nuance lives in the entry's `status`.

**Example:** an entry with `status: in-progress` has an unchecked `- [ ]` on the board. This is correct — the checkbox tracks done/not-done, while `status` captures the finer state.

### Move or rename a task

```sh
wiki move --dry-run /3-graph/001-parser.md /3-graph/010-parser.md
# confirm, then:
wiki move /3-graph/001-parser.md /3-graph/010-parser.md
```

Rewrites the board's link and any other inbound links in one step. Update the board text if the title changed.

### Check dependencies and impact

Before starting or closing a task, check what references it and what it depends on:

```sh
wiki backlinks /3-graph/010-parser.md     # which entries link to this task?
wiki unresolved                           # are there linked specs or designs that don't exist yet?
```

Use `backlinks` to assess impact: if many entries reference a task, its completion may unblock other work. Use `unresolved` to surface missing prerequisites — linked design docs or specs that haven't been written yet are candidates for blocking items.

### Groom the base

```sh
wiki check                 # health report (warnings exit 0; errors exit 1)
wiki check --fix           # safe auto-repairs
wiki tidy                  # preview canonicalization
wiki tidy --all            # apply: root-absolute links, slug filenames
```

**After any batch of edits, run `wiki check` before committing.**

### Prune completed items

The board is a short-lived working surface. The task entry is a working record: once work ships, its lasting value is captured elsewhere and the file can go. Prune completed items that no longer earn their place:

| Category | Board checkbox line | Task entry file | Permanent record |
|---|---|---|---|
| **Routine** (small fix, chore) | `- [x]` for a few sprints, then remove | Kept as `status: done` for a few sprints, then delete | — |
| **Milestone** (notable, audit-worthy) | Remove | Delete | Dated entry in `log.md` with outcome and key decisions |
| **Superseded / merged** (obsolete, rolled into another) | Remove | Delete | Brief note in `log.md`: what replaced it and why |

When a task is superseded, merged, or obsolete, delete the file and its board line in the same change. Note the superseding in `log.md` (if present and relevant) and in the survivor entry.

**If a milestone's outcome deserves to live in the broader KB** (not just as a log note, but as a searchable concept or decision record) create a new KB entry and link from `log.md`. Do not keep the task file for this purpose; promote relevant knowledge, discard the working record.

### Improve linking while grooming

When reviewing task entries, look for concepts, tools, or other tasks mentioned in the body that are not yet linked. Add root-absolute links to related entries. A task that references "[Docker](/tech/infra/docker.md)" is more navigable than one that just mentions "Docker." Cross-linking turns a flat list into a navigable graph.

### Persisting to git

Git is optional. When present:

```sh
git pull --rebase       # always before editing
git add -A && git commit -m "summary here" && git push
```

Always pull upstream before editing. On merge conflicts, resolve them yourself: understand and preserve the intent of both sides, merge frontmatter fields sensibly. Only escalate to the user if the conflict requires a judgment you cannot make.

## Conventions

- **Single source of truth:** board checkbox checked exactly when entry `status` is `done`.
- **Board never goes stale:** update the board in the same change as the status.
- **Stay green:** `wiki check` passes. Warnings are informational: address when practical.
- Root-absolute links (`/domain/file.md`), no wikilinks. Broken links are tolerated — they are future knowledge.
- Reserved files `index.md` / `log.md` carry no frontmatter, are exempt from `type`, and skipped by `wiki orphans`. The root `index.md` may carry `okf_version`. `log.md` is optional — create it only when a domain benefits from an audit trail.
- **Slug names for folders and files:** lowercase, dash-separated words, no spaces, no underscores. Folder structure is the agent's judgment.
- **Timestamp:** set `timestamp` (ISO 8601) when relevant, and when an entry meaningfully changes.
- Technical debt may have its own section/folder (`tag: debt`).

## LEARNINGS.md

Maintain a `LEARNINGS.md` at the root of the backlog. It is a self-improvement log for the agent — not a user-facing document. Whenever you notice a gap, inconsistency, pattern, or improvement opportunity during grooming, capture it:

```markdown
## 2026-06-28

- Board checkbox for `001-parser.md` is unchecked but entry status is `done` — sync.
- Tasks in `/3-graph/` all reference "the spec" but no `/3-graph/spec.md` exists — create it.
- Tags `frontend` and `fe` both used — consolidate.
```

## Exit codes

`wiki` mirrors the tools it resembles. `0` is success: `tasks`, `list`, `property`, and the other query commands return `0` even with no results (like `ls`). Exit `1` comes from just two commands: `search` with no match (like `grep`) and `check` with conformance errors (like a linter). Exit `2` is a real error (no bundle, bad arguments): investigate.

## CLI Tool

If the `wiki` is not installed on this system, refer to https://raw.githubusercontent.com/agentic-wiki/wiki/refs/heads/main/README.md

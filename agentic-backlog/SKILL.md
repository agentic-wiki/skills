---
name: agentic-backlog
description: Manage a task backlog — see what is next, pick up work, mark it done, add tasks, add context, keep the board fresh. Use when a project has a non-trivial backlog and a wiki bundle is the source of truth.
---

# Managing a task backlog

**Goal:** keep a backlog that is always an honest, lean snapshot. The board answers "what's next" truthfully, every task entry is a durable record of what shipped and why, and the board never bloats with stale completions.

The backlog is a wiki bundle: plain markdown files. Each task is a `type: task` entry; the work-kind (`feature`/`bug`/`debt`/`chore`) is a **tag**; the board is `index.md` (checkboxes linking to the entries).

The quality of any query (`wiki tasks`, `wiki list --type task`, `wiki search`) is only as good as the indexing done before it. A checklist that says "what's next" is truthful only if the checkboxes, statuses, and links were kept current.

The mechanical work of querying, checking integrity, and rewriting links when moving is where `wiki` helps. The judgment work — what to index, how to classify, when to prune — is yours.

## Use cases

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

4. Verify:

```sh
wiki check
```

### Updating task status

Set the entry's frontmatter `status: todo | in-progress | blocked | done`.

### When a task is done

In a single change so the board never goes stale:

1. Set the entry's frontmatter `status: done`.
2. Flip the board checkbox to `- [x]` (or move under `## Done`).
3. You may append a short `Done:` line to the entry if useful: what shipped (if different), plus any decision worth keeping.

**For milestones or anything worth keeping after the board is pruned, also append to the phase's `log.md`** under an ISO date heading:

```markdown
## 2026-06-28

- [010-parser](/3-graph/010-parser.md) — shipped the link graph parser. Dropped the nomnom approach, used bufio instead.
```

`log.md` is the lasting audit trail; the board is a short-lived working surface.

If you find a board checkbox and entry `status` out of sync, correct both: the checkbox is checked exactly when `status` is `done`.

### Richer statuses

`status` is a free field: `todo`, `in-progress`, `blocked`, `done`. The board checkbox stays binary — `in-progress` and `blocked` tasks remain `- [ ]` and appear in `wiki tasks`. The nuance lives in the entry's `status`.

### Move or rename a task

```sh
wiki move --dry-run /3-graph/001-parser.md /3-graph/010-parser.md
# confirm, then:
wiki move /3-graph/001-parser.md /3-graph/010-parser.md
```

Rewrites the board's link and any other inbound links in one step. Update the board text if the title changed.

### Groom the base

```sh
wiki check                               # health report (warnings exit 0; errors exit 1)
wiki check --fix                         # safe auto-repairs
wiki tidy                                # preview canonicalization
wiki tidy --all                          # apply: root-absolute links, slug filenames
```

**After any batch of edits, run `wiki check` before committing.**

### Prune completed items

A board's `## Done` grows without bound. At the end of big phases, prompt the user to prune completed items that no longer earn their place. What goes where:

| What | Stays in board | Moves to `log.md` | Both deleted |
|---|---|---|---|
| Routine tasks (small fix, chore) | `- [x]` for a few sprints | — | then removed |
| Milestones, audit-worthy work | — | date-entry in `log.md` | entry file remains |
| Superseded / merged tasks | — | "001-parser superseded by 005-graph" | deleted file + board line |

When a task is superseded, merged, or obsolete, delete the file and its board line in the same change. Note the superseding in `log.md` and in the survivor entry.

### Improve linking while grooming

When reviewing task entries, look for concepts, tools, or other tasks mentioned in the body that are not yet linked. Add root-absolute links to related entries. A task that references "[Docker](/tech/infra/docker.md)" is more navigable than one that just mentions "Docker." Cross-linking turns a flat list into a knowledge graph.

### Persisting to git

Git is optional. When present:

```sh
git pull --rebase       # always before editing
git add -A && git commit -m "summary here" && git push
```

Always pull before editing. On merge conflicts, resolve them yourself: understand and try to preserve the intent of both sides, merge frontmatter fields sensibly. Only escalate to the user if the conflict requires a judgment you cannot make.

## Conventions

- **Single source of truth:** board checkbox checked exactly when entry `status` is `done`.
- **Board never goes stale:** update the board in the same change as the status.
- **Stay green:** `wiki check` passes. Warnings are informational — address when practical.
- **Slug names for folders and files:** lowercase, dash-separated words, no spaces, no underscores. Folder structure is the agent's judgment.
- **Timestamp:** set `timestamp` (ISO 8601) when an entry meaningfully changes.

## Exit codes

`wiki` exits `0` when results are found, `1` when no results match (not an error), and `2` on actual errors. Treat exit `1` as normal "none found."

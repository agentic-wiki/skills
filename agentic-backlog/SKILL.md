---
name: agentic-backlog
description: Operate a wiki-format task backlog with the `wiki` CLI; see what is next, pick up work, mark it done, add tasks, add context, keep the board fresh. Use when a project has a non trivial backlog and a wiki bundle can solve the problem.
---

# Operating a task backlog with `wiki`

**Goal:** keep a backlog that is always an honest, lean snapshot. `wiki tasks` answers "what's next" truthfully, every task entry is a durable record of what shipped and why, and the board never bloats with stale completions.

The backlog is an agentic wiki bundle: `tasks/` holds `wiki.toml` and the task entries directly inside it. Each task is a `type: task` entry; the work-kind (`feature`/`bug`/`debt`/`chore`) is a **tag**; `tasks/index.md` is the board (checkboxes linking to the entries). Run `wiki` inside `tasks/`.

## See what is next

- `wiki tasks`: open board items (the working "what's next").
- `wiki list --type task`: every task entry, regardless of status.
- `wiki list --type task --tag debt`: filter by kind (`debt`, `feature`, …).
- `wiki list --type task --prefix 3-graph-and-mutation/`: filter by phase.
- `wiki property status --counts`: todo/done at a glance. `wiki tags --counts`: work by kind.
- `wiki check`: confirm the bundle is healthy (links resolve, every entry typed). Run it before and after edits.

## Pick up a task

Choose one from `wiki tasks` (scope a big board with `--prefix <folder>/`), then open its linked entry for the full context and fields (`priority`, `due`). A task entry may hold its own inline `- [ ]` steps; `wiki tasks` surfaces every checkbox, entries and boards alike.

## Mark a task done

In a single change, so the board never goes stale:

1. Set the entry's frontmatter `status: done`.
2. Flip its board checkbox `- [ ]` to `- [x]` (or move it under a `## Done` section).
3. Append a short `Done:` line to the entry: what shipped, plus any decision worth keeping (an option you dropped, a trade-off). The entry is the durable record; keep the board line itself terse.

Then `wiki tasks` should no longer list it, while `wiki list --type task` still shows the entry.

## Richer statuses

`status` is a free field; a useful set is `todo`, `in-progress`, `blocked`, `done`. The board checkbox stays binary, so an `in-progress` or `blocked` task is still an open `- [ ]` and still appears in `wiki tasks`; the nuance lives in the entry's `status` (`wiki property status --counts` shows the spread). For a `blocked` task, record what it is waiting on and what unblocks it.

## Add a task

1. Create `tasks/<phase>/<NNN>-slug.md` with frontmatter: `type: task`, `status: todo`, a `priority`, and a kind tag (`tags: [feature]` or `[debt]`). Zero-pad `<NNN>` so inserting never forces a renumber.
2. Add a checkbox to the board linking to it with a root-absolute path: `- [ ] [Title](/<phase>/<NNN>-slug.md)`.
3. `wiki check` to confirm the link resolves and the type is known.

## Move or renumber a task

`wiki move /<phase>/<old>.md /<phase>/<new>.md` relocates or renumbers a task entry and rewrites the board's link to it (and any other links) in the same step, so nothing dangles. Update the board text afterward if the title changed.

## Keep the backlog lean

Removing matters as much as adding.

- Before adding, `wiki search "<query>"` (a distinctive term from the title; the match is literal substring, so prefer one word over a phrase) to avoid duplicating an existing task.
- When a task is finished-and-superseded, merged into another, or no longer relevant, delete the file and its board line in the *same* change, and note it in the survivor ("Supersedes X"). A stale open item is worse than none.
- **Prune old completed items.** A board's `## Done` grows without bound, and `index.md` becomes dead weight you pay for every time. This matters as much as keeping each done line terse: **at the end of big phases prompt the user to prune completed items that no longer earn their place in the board.** Nothing is lost, the task files can remain and `wiki list --type task` still shows them, so the entries may be the archive. (If a lasting changelog was needed, rather than a giant Done list, append shipped milestones to a dated `log.md`.)

## Conventions

- **Single source of truth:** the board checkbox is checked exactly when the entry's `status` is `done`; other statuses (`in-progress`, `blocked`) keep it open.
- **Board never goes stale:** update the board in the same change as the status.
- **Stay green:** `wiki check` passes (no broken links, every entry typed).
- **Persist together:** if the backlog belongs in a git repo; offer to commit when a round of changes is done, and sync with `git pull --rebase` before pushing.
- **Slug filenames:** lowercase, hyphenated, no spaces.
- **Timestamp:** set `timestamp` (ISO 8601, e.g. `2026-06-24`) when an entry meaningfully changes; `check` errors only on a malformed one, never on its absence.
- Put enough context in each task file for a future developer, without bloat.

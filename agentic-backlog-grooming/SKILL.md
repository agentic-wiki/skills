---
name: backlog-grooming
description: Cron-triggered AI agent skill for grooming the personal wiki backlog. Runs periodic health checks on tasks: detects stale items in soon/, identifies blocked tasks, flags orphans, and proposes moves between timeboxes (now/soon/someday/archive). Use when the backlog needs maintenance, triage, or a periodic review. Triggers on "groom backlog", "backlog maintenance", "stale tasks", or scheduled cron execution.
---

# Backlog Grooming Skill

**Goal:** keep the backlog honest and lean. Items in `now/` are truly next; `soon/` is not a graveyard; blocked items get reviewed; stale undone work goes to `archive/`.

Run `wiki` inside the appropriate bundle root.

## Backlog structure

The backlog is time-first, scope-tagged:

```
backlog/
├── index.md              # board: Now / Soon / Someday columns, scope tags inline
├── now/                  # this week — small set, truly next
├── soon/                 # next 1-4 weeks
├── someday/              # open-ended, no deadline pressure
├── archive/              # undone items that aged out of soon/someday without attention
└── done.md               # completed items log (optional, lightweight)
```

Each task file has frontmatter: `type: task`, `status: todo|in-progress|blocked|done`, `priority: low|medium|high`, `tags: [...]`.

The board (`index.md`) is organized by time column, not by scope. Scope appears as an inline tag/label on each line.

## Grooming procedure

Run these steps in order. Produce a **grooming report** — don't silently move things. Present findings to the user for approval before making changes.

### 1. Health check

```sh
wiki check --prefix /backlog/
wiki tasks --all --prefix /backlog/
```

Note any broken links, missing types, or status/checkbox mismatches.

### 2. Age analysis — stale items in `soon/`

List all tasks in `soon/` with their timestamps:

```sh
wiki list --type task --prefix /backlog/soon/ --sort=timestamp
```

**Rule:** any task in `soon/` older than **14 days** with no status change is a candidate for:
- Move to `someday/` if it's still valid but not urgent
- Move to `archive/` if it seems abandoned or no longer relevant

Flag each one in the report with its age and suggested destination.

### 3. Blocked review

Find all blocked tasks across all timeboxes:

```sh
wiki list --type task --prefix /backlog/ --format json | grep -i blocked
```

For each blocked task:
- How long has it been blocked?
- Is the blocking condition still valid?
- Suggest: unblock, defer to `someday/`, or archive

### 4. Orphan detection

```sh
wiki orphans --prefix /backlog/
```

Task files not linked from `index.md` are unindexed — they won't show in `wiki tasks`. Either add them to the board or delete them if stale.

### 5. `now/` overflow check

```sh
wiki list --type task --prefix /backlog/now/
```

**Rule:** `now/` should have **≤ 5 items**. If it has more, suggest which ones move to `soon/`.

### 6. Archive candidates

Scan `someday/` for items older than **60 days**:

```sh
wiki list --type task --prefix /backlog/someday/ --sort=timestamp
```

Items this old that haven't moved to `soon/` or `now/` are likely forgotten. Suggest moving to `archive/`.

### 7. Cross-reference check

```sh
wiki unresolved --prefix /backlog/
```

### 8. Structural soundness

Identify when the structure is becoming hard to consume and take small steps to improve clarity over time. This compounds over time.

## Scheduling

This skill is designed to run via cron. A typical schedule:

```cron
# Every Monday at 09:00
0 9 * * 1 cd /home/me/wiki-bundle && <trigger-agent> --skill backlog-grooming
```

The agent invocation depends on your setup. The skill itself is the instructions the agent follows when triggered.

## Conventions

- Use `wiki move` for all relocations — it rewrites links atomically
- Keep the board (`index.md`) in sync with file locations
- Don't delete task entries without user approval — move to `archive/` instead
- Update the board's checkbox text if the task title changed
- After all moves, run `wiki check` to verify integrity

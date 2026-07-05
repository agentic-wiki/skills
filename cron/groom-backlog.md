## Groom the task backlog

Run a periodic health check over the backlog (a wiki bundle of `type: task` entries) and produce a **grooming report**. Propose structural moves, don't apply them silently: present findings for approval, then act.

Scope the commands to the backlog with `--prefix` if it lives under a subfolder (e.g. `--prefix /backlog/`).

1. **Health check.** `wiki check` and `wiki tasks --all`. Note broken links, missing types, and status/checkbox mismatches (a checkbox is checked exactly when its entry's `status` is `done`).
2. **Stale work.** `wiki list --type task --sort=timestamp` (add `--reverse` for oldest first). Flag tasks that have sat untouched too long: still valid but not urgent, or seemingly abandoned. Suggest a destination for each.
3. **Blocked review.** Find `status: blocked` tasks (`wiki list --type task --format json`; `wiki property status --counts` for the tally). For each: how long has it been blocked, is the blocker still real, and should it be unblocked, deferred, or dropped.
4. **Orphans.** `wiki orphans` — task files nothing links to won't appear on the board or in `wiki tasks`. Add them to `index.md` or remove them if stale.
5. **Board overflow.** Scan the board's active set. If "what's next" has grown beyond what's realistic for the period, suggest which items to defer.
6. **Cross-references.** `wiki unresolved` — linked specs or designs that don't exist yet are candidate prerequisites or blocking items.
7. **Structural soundness.** If the layout is getting hard to consume, propose one small step to improve clarity. This compounds over time.

Use `wiki move` for every relocation (it rewrites links atomically), keep the board in sync with file locations, and run `wiki check` after all changes. Don't delete task entries without approval; prefer moving them out of the active set.

If the backlog is organized by timebox, the same checks apply per column: `now/` stays small and truly next, `soon/` doesn't become a graveyard, and items that age out of `soon/`/`someday/` without attention move to `archive/`. That layout is one option, not a requirement: the folder structure is the agent's judgment.

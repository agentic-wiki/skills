---
name: agentic-wiki
description: The user's knowledge base is the source of truth for everything they know and are working on. Use whenever they jot or brain-dump a thought or idea, ask what they know or where something is, look up / recall / search their notes, add / organize / link / refile entries, ask what to think about next, or groom and maintain the base. Triggers on any mention of their knowledge base, wiki, vault, notes, store, or second brain. Operates an agentic-wiki bundle (markdown + wiki.toml) with the `wiki` CLI. For a task backlog specifically, agentic-backlog goes deeper.
---

# Operating a knowledge base with `wiki`

**Goal:** make the knowledge base the reliable source of truth for what the user knows and is working on. Capture without friction, file precisely, recall fast, groom regularly, and never lose an entry or leave a link dangling. The files are the user's forever; you are the librarian.

The base is a folder marked by `wiki.toml`: every `.md` file is an entry with YAML frontmatter, linked by root-absolute paths (`/domain/file.md`). Run `wiki` inside it (it walks up to find `wiki.toml`) or `wiki --root <dir> …`.

**For the complete, current commands and flags, run `wiki --help` and `wiki <command> -h`.** This skill is the judgment layer: when to reach for what, the lifecycle, the typical moves, and the conventions, not a copy of the reference (which would drift).

## The model

- **Folder is one stable home**, by domain (`finance/`, `tech/infra/`); relocate with `wiki move` (it rewrites every link).
- **`type` is what kind** it is (`note`, `concept`, `dataset`, `task`, … from `wiki.toml`), required on every entry; `draft` is the "not yet classified" type for raw captures.
- **Tags are everything cross-cutting** (`database`, `2026`, `needs-review`).

If a thing would ever live in two folders at once, that second axis is a tag, not a folder.

## The lifecycle

Six stages. They chain across sessions; you rarely run them all at once. Adding is as first-class as reading: a casual brain-dump and a good answer matter equally.

- **Capture** (get it down, decide nothing). Jot a rough thought as a `type: draft` in `inbox/`; that queue is `wiki list --type draft`. For a file, drop it in the gitignored `inbox/resources/` and point the draft's `resource:` at it.
  _Example:_ create `inbox/cheap-flights-idea.md` with `type: draft` and the raw note.
- **Refine** (draft into a real entry). Read the draft back, ask the user the one or two questions that sharpen it, fill it in. Summary first, detail after.
  _Example:_ `wiki read /inbox/cheap-flights-idea.md`, then expand it.
- **Organize** (file it into the graph). Set the real `type`, move it into its domain, and link it from the domain `index.md` and related entries so it is never an orphan.
  _Example:_ `wiki move /inbox/cheap-flights-idea.md /personal/travel/cheap-flights.md`, then link it in.
- **Query / retrieve** (find anything). Start top-down from `index.md` and follow links, or target it:
  - `wiki list --type concept --tag crypto --prefix tech/` (by kind, tag, subtree)
  - `wiki search "language model" --lines` (free text; file:line)
  - `wiki backlinks <path>` (what points here), `wiki unresolved` (promised but unwritten), `wiki orphans` (nothing links in)
  - `wiki property status`, `wiki tags`, `wiki properties` (what values and vocabulary exist)
  - `wiki read`/`outline` (an entry's body / its headings)
- **Groom** (keep it healthy). `wiki check` (broken links, missing or unknown types, version drift), `wiki tidy` (canonicalize links and slug filenames), refile with `wiki move`. Periodically audit a stale or random entry and fix it; `wiki unresolved` is a standing to-write list.
- **Persist** (it is a git repo). `git pull --rebase`, commit, push. `inbox/resources/` (binaries) and `.wiki/` (cache) are gitignored; drafts and entries are committed.

## Good practices

- **One entry, one thing.** A sharp entry is findable and linkable; split a page that wants two titles.
- **Summary first, detail after.** The opening sentences are what retrieval and an LLM read first.
- **Link as you write.** An unlinked entry is lost knowledge; wire it into an `index.md` and its relatives immediately.
- **Navigate top-down, then target.** Read `index.md` and follow links before reaching for `search`.
- **Decide structure at refine time, not capture.** Capturing must be frictionless; kind, domain, and tags come later.
- **Let the tool move things.** Always `wiki move` / `wiki tidy`; never hand-edit a path or a link.
- **Broken links are a backlog, not a bug** (`wiki unresolved` is the list).
- **Keep it human-greppable.** Real Markdown, slug filenames, shallow folders: someone with no tools should still navigate it.

## Conventions

- Every entry carries a `type` (`draft` until classified). Reserved `index.md`/`log.md` carry no frontmatter (the root `index.md` may carry `okf_version`).
- Root-absolute links (`/domain/file.md`), no wikilinks. Broken links are tolerated.
- Slug filenames: lowercase, hyphenated, no spaces.
- Shallow folders: two levels by default (`wiki check` warns past three).
- Keep `wiki check` green before each commit.

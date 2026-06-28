# agentic-wiki skills

Operating manuals an AI agent follows to drive the [`wiki` CLI](https://github.com/agentic-wiki/wiki) over an [agentic-wiki](https://github.com/agentic-wiki/spec) knowledge base. The format and the tool stand on their own; these skills are an implementation layer, which you can tailor to your use case.

- **[agentic-wiki](agentic-wiki/SKILL.md):** operating a knowledge base (capture, refine, promote, index, retrieve, maintain).
- **[agentic-backlog](agentic-backlog/SKILL.md):** operating a task backlog that is itself a wiki bundle.

They follow the open Agent Skills standard (a `SKILL.md` with `name`/`description` frontmatter), so the same files work in all compatible agents.

## Install

These are starting points to copy and customize, not pinned dependencies. Pick whichever fits:

**Let your agent install it.** Tell your agent:

> Install the skill at `https://raw.githubusercontent.com/agentic-wiki/skills/main/agentic-wiki/SKILL.md` into your skills directory, then help me adapt it.

Your agent knows its own runtime's skills directory, so one instruction covers Claude Code, Codex, Pi, Hermes, and the rest, and it can customize the skill with you on the spot.

**Do it by hand.** Each skill is one `SKILL.md` in its own folder; drop the folder into your runtime's skills directory:

| Runtime | Project | Global |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.codex/skills/` | `~/.codex/skills/` |
| Others (Pi, Hermes, …) | their `.<runtime>/skills/` equivalent | |

```sh
cp -r agentic-wiki agentic-backlog ~/.claude/skills/   # adjust the dir per runtime
```

**Fork the lot.** `git clone` this repo, copy the folders you want, and version your own changes.

Then edit freely: these are conventions you make yours.

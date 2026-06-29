# Agentic Wiki skills

Operating manuals an AI agent follows to drive the [`wiki` CLI](https://github.com/agentic-wiki/wiki) over an [agentic-wiki](https://github.com/agentic-wiki/spec) knowledge base. The format and the tool stand on their own; these skills are an implementation layer, which you can tailor to your use case.

They follow the open Agent Skills standard (a `SKILL.md` with `name`/`description` frontmatter), so the same files work in all compatible agents.

## [Agentic Wiki](agentic-wiki/SKILL.md)

Operates a knowledge base (capture, refine, promote, index, retrieve, maintain) living in a standard wiki bundle using the `wiki` CLI.

Install the skill by pasting the following text to your agent of choice:

> [!NOTE]
> Install the skill at https://raw.githubusercontent.com/agentic-wiki/skills/main/agentic-wiki/SKILL.md into your skills directory.

## [Agentic Backlog](agentic-backlog/SKILL.md)

Operates a task backlog living in a standard wiki bundle using the `wiki` CLI.

Install the skill by pasting the following text to your agent of choice:

> [!NOTE]
> Install the skill at https://raw.githubusercontent.com/agentic-wiki/skills/main/agentic-backlog/SKILL.md into your skills directory.

## Install details

These skills are just starting points to copy and customize. Tell your agent to adapt them if your workflow needs additional guidance.

**Agent runtimes and their skills directories:**

| Runtime | Project-local | Global |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Codex | `.codex/skills/` | `~/.codex/skills/` |
| Others (Pi, Hermes, …) | their `.<runtime>/skills/` equivalent | |

**Fork the lot.** `git clone` this repo, copy the folders you want, and version your own changes.

Then edit freely: these are conventions you make yours.

## Periodic checks ("cron" jobs)

Beyond the interactive skills above, you can ask your agent to run lightweight, scheduled checks on its own initiative. These are not skills: they are recurring prompts or reminders you set up outside the agent (a calendar reminder, a cron task that calls the agent, a recurring message in your chat). The agent runs them against the base and reports findings.

- **[learnings](cron/learnings.md)**: review `LEARNINGS.md`, group by theme, separate quick fixes from decisions.
- **[spot-check](cron/spot-check.md)**: pick 5–10 entries at random and check for grooming issues.
- **[new-ideas](cron/new-ideas.md)**: scan the base for patterns, gaps, or connections that suggest directions worth exploring.

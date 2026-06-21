# Operon Skills

Skills that let an AI assistant (Claude Code, Codex, Cursor, etc.) drive
the [Operon](https://withoperon.com) CLI on a user's behalf — install,
mine sessions, polish, publish.

## Install

Add a skill to your local agent skills store via the [`npx skills`](https://skills.sh) CLI:

```bash
# Onboard a fresh user end-to-end
npx skills add Try-Operon/skills --skill operon-setup

# Help an existing user publish one of their drafts
npx skills add Try-Operon/skills --skill operon-publish

# Run a fresh mining pass on a recurring cadence
npx skills add Try-Operon/skills --skill operon-mine

# Search the team catalog for a shared operon
npx skills add Try-Operon/skills --skill search-operons

# Install a specific operon into your local agents
npx skills add Try-Operon/skills --skill install-operon

# Or add all of them at once
npx skills add Try-Operon/skills
```

Each command drops a `SKILL.md` under `~/.claude/skills/<name>/`
(and the equivalent path for Codex/Cursor).

## Skills in this repo

| Skill | What it does |
|---|---|
| [`operon-setup`](./operon-setup/SKILL.md) | One-shot first-run: install the CLI, sign in via the browser, detect AI tools, pick a workspace, mine the first candidates. |
| [`operon-publish`](./operon-publish/SKILL.md) | Walks the user through picking one of their drafts and shipping it to their team. |
| [`operon-mine`](./operon-mine/SKILL.md) | Run a fresh mining pass against the user's Claude Code / Codex / Cursor sessions. Recurring loop. |
| [`search-operons`](./search-operons/SKILL.md) | Search the Operon catalog for shared operons published by engineering teams. |
| [`install-operon`](./install-operon/SKILL.md) | Install a specific operon (versioned AI skill) into Claude Code, Codex, and Cursor. |

## Alternate install path

Operon also publishes these skills directly via the [Cloudflare
Agent-Skills Discovery RFC v0.2.0](https://github.com/cloudflare/agent-skills-discovery-rfc)
at:

```
https://withoperon.com/.well-known/agent-skills/index.json
```

Each entry has a `sha256` digest that matches the file in this repo.

## What is Operon?

Operon is a team layer for the AI workflows your engineers already
run. The CLI scans Claude Code, Codex, and Cursor sessions on each
laptop (local-first, redaction on disk), surfaces recurring prompts,
and publishes them to your team as installable **operons** — versioned
skills any teammate can `operon install <workspace>/<slug>`.

Read more at [withoperon.com](https://withoperon.com).

## Contributing

The source of truth for each skill lives in the Operon platform repo
under `apps/landing/public/.well-known/agent-skills/`. This repo is a
public mirror that the `npx skills` ecosystem can install from.

Open a PR here for typos / clarifications and we'll sync the change
back to the source. For larger changes (a new skill, a behavior
change), open a discussion at
[withoperon.com](https://withoperon.com) first.

## License

MIT — see [LICENSE](./LICENSE).

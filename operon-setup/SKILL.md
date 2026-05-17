---
name: operon-setup
description: Walk a user through installing Operon and publishing their first AI skill, end-to-end. Drives the `operon` CLI on the user's behalf.
type: skill
version: 0.1.0
---

# Operon — Setup

A meta-skill: install the Operon CLI on a fresh machine, authenticate
the user via a browser tab, detect their AI tools, pick or create a
workspace, mine their first candidates, and surface the top three so
they can publish one. Designed to be driven by an AI assistant
(Claude Code, Codex, Cursor) end-to-end.

## When to use this skill

- The user says "set up Operon for me" or "install Operon."
- The user has a recurring prompt they keep retyping in Claude Code,
  Codex, or Cursor and asks how to share it with a teammate.
- The user wants to see what AI workflows recur across their sessions
  and doesn't know where to start.

## How to use it

### 1. Check prerequisites

```
node --version
```

Require node ≥ 22. If older, point them at https://nodejs.org and stop.

### 2. Install the CLI

```
npm i -g @withoperon/cli
```

If they're on a system without global npm permissions, suggest the
`pnpm dlx @withoperon/cli` form instead.

### 3. Run the unified setup verb

```
operon setup --yes --json
```

Three important things about this invocation:

- **`--yes`** accepts every default that has one. Workspace name will
  default to the user's `$USER`; mining window defaults to 30d.
- **`--json`** switches the CLI into NDJSON mode — one JSON object per
  line is written to stdout the instant each step completes. **Parse
  stdout line by line** instead of waiting for one final blob.
- The CLI opens the user's browser to a URL like
  `https://app.withoperon.com/auth/cli?code=ABC12345`. They click
  Approve. No copy/paste of tokens.

### 4. Parse NDJSON events as they land

Each event has the shape `{ ts, cmd, level, event, data }`. Watch for:

| `event` | Meaning | What to do |
|---|---|---|
| `started` | Setup began | Show a "Setting up Operon…" status |
| `auth.login_started` | Device-code flow started | Tell the user "Your browser will open in a second" |
| `auth.ready` | User signed in | Note `data.apiBase` |
| `sources.detected` | Claude Code / Codex / Cursor presence | Read `data.sources` (array of names) |
| `workspace.picked` | Workspace selected/created | Save `data.slug` for downstream commands |
| `mine.complete` | First mine finished | `data.candidates` is the array — present top 3 to user |
| `done` | Setup is complete | Move to step 5 |

If the user closes the terminal mid-setup, you can recover state with:

```
cat ~/.operon/events.log | tail -50
```

### 5. Present the top candidates

After `mine.complete`, you have a list like:

```json
{
  "slug": "review-pr",
  "score": 0.91,
  "uses": 12,
  "summary": "Review the diff for security, performance, and consistency issues."
}
```

Show the user the top 3 in plain English and ask which to publish.

### 6. (Optional) Polish the chosen candidate with Claude

Only if `ANTHROPIC_API_KEY` is set in the environment:

```
operon polish <slug>
```

This drafts a clean `SKILL.md` from the raw candidate. Skip if the key
isn't set — `operon publish` works without polishing.

### 7. Publish

```
operon publish <slug>
```

Report the resulting URL back to the user (it'll look like
`https://app.withoperon.com/<workspace>/<slug>`).

## Error handling

If a command exits non-zero, parse stderr for the line
`https://withoperon.com/docs/cli/errors#<slug>`. The error code prefix
tells you the recovery step:

- **`AUTH_REQUIRED`** → re-run `operon setup`. Their token is missing
  or invalid.
- **`NO_SOURCES`** → tell the user to install Claude Code, Codex, or
  Cursor first. Operon needs at least one to mine.
- **`NO_CANDIDATES`** → retry with `--since 90d` or `--since 180d`.
  Their sessions don't have enough repeats yet in the current window.
- **`WORKSPACE_NOT_LINKED`** → re-run `operon setup` (or pass
  `--workspace <slug>` if they know the slug).
- **`RATE_LIMITED`** → wait a minute and retry.
- **`NETWORK_ERROR`** → check connectivity; if they self-host, pass
  `--api-base <url>`.
- **`USER_ABORT`** → exit code 0; they canceled. Re-run when ready.

## Strict (CI / non-interactive) mode

If you're driving the CLI in a fully-automated context where no human
will see prompts, also pass `--non-interactive`. The CLI will fail fast
with a clear error code if any input was needed instead of hanging.

```
operon setup --yes --json --non-interactive
```

## What setup does NOT do

- Doesn't install Claude Code, Codex, or Cursor for the user. Those are
  prerequisites; if none are present, setup aborts with `NO_SOURCES`.
- Doesn't auto-publish anything. The mine step is `--dry-run` — the
  user explicitly opts in via `operon publish`.
- Doesn't read session content beyond what Operon itself needs to
  cluster recurring prompts. All mining is local-first; nothing leaves
  the laptop until publish.

## Notes for agents

- The CLI writes its config to `~/.operon/config.json`. After the
  first successful `operon setup`, subsequent runs are authenticated
  automatically — you don't need to repeat the device-code dance.
- If you see a `started` event but no `auth.ready` after ~30 seconds,
  the user hasn't clicked Approve yet — surface the URL from
  `auth.browser_opened.data.url` so they can find the tab.
- The setup verb is idempotent. Calling it twice does the right thing:
  step 1 short-circuits if already authed, step 3 short-circuits if a
  workspace is already linked, step 4 short-circuits if candidates
  already exist on disk.

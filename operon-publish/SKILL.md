---
name: operon-publish
description: Help a user pick one of their mined drafts and ship it to their team via `operon publish` (or the web Drafts dashboard).
type: skill
version: 0.1.0
---

# Operon — Publish a Draft

After `operon mine` runs, each detected pattern lands on the user's
disk under `~/.operon/candidates/<slug>/` **and** uploads to their
workspace's private Drafts slot at
`https://app.withoperon.com/<workspace>/candidates`. This skill walks
an AI assistant through helping the user pick one of those drafts and
publish it to their team.

## When to use this skill

- The user says "publish my best prompt" or "ship this one to my team."
- They've just run `operon mine` (or `operon setup`) and are looking at
  3–5 candidates wondering which to publish.
- They opened the web Drafts tab and want a recommendation.

## How to use it

### 1. Confirm the user is set up

```
operon doctor --json
```

If `auth` is missing, run `operon setup` first. If `sources` is empty,
the user doesn't have Claude Code / Codex / Cursor installed yet —
stop and tell them.

### 2. List the user's drafts

Two equivalent paths — pick whichever fits the conversation:

**CLI:**
```
operon list --json
```

Output is an array of candidate summaries with `slug`, `score`,
`uses`, `spanDays`. Drafts also live on the platform at
`https://app.withoperon.com/<workspace>/candidates` — surface that
URL to the user if they want to read each draft in a browser.

### 3. Show the top 3 to the user and ask

Sort by `score` descending and show:

```
1. review-pr     (score 0.91, 12 uses across 14 days)
2. migrate-down  (score 0.84,  8 uses across  9 days)
3. ship-hotfix   (score 0.77,  6 uses across 11 days)
```

Ask which one to publish. Don't pick for them — these are their
prompts, they know which one feels real.

### 4. (Optional) Polish before publishing

The mined draft is rough — auto-extracted from session content. If
the user has `ANTHROPIC_API_KEY` set, offer to refine it:

```
operon polish <slug>
```

This rewrites the draft using Claude with the original exemplar
sessions as context (those bodies stay on the user's laptop).

If `ANTHROPIC_API_KEY` isn't set, you can polish from the web Drafts
dashboard instead — there's a "Polish with Claude" button that uses
Operon's platform key. Surface the URL:

```
https://app.withoperon.com/<workspace>/candidates/<slug>
```

### 5. Publish

```
operon publish <slug>
```

Report the resulting URL back to the user. It'll look like:

```
https://app.withoperon.com/<workspace>/<slug>
```

Their teammates can now `operon install <workspace>/<slug>` to drop
it into their own Claude Code / Codex / Cursor.

## Error handling

If the publish step fails, parse stderr for the
`https://withoperon.com/docs/cli/errors#<slug>` link:

- **`AUTH_REQUIRED`** → re-run `operon login`.
- **`WORKSPACE_NOT_LINKED`** → run `operon setup` or pass
  `--workspace <slug>` to publish.
- **`RATE_LIMITED`** → wait a minute and retry.
- **`NETWORK_ERROR`** → check connectivity.

## Notes for agents

- Publishing creates a versioned operon. The first publish lands at
  v0.1.0; subsequent publishes auto-bump the patch.
- Visibility defaults to `team`. To publish privately (only the miner
  sees it), pass `--visibility private`.
- The on-disk draft at `~/.operon/candidates/<slug>/` is left alone
  after publish — the user can re-mine over it any time.

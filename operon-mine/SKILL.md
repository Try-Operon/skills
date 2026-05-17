---
name: operon-mine
description: Run a fresh mining pass across a user's Claude Code, Codex, and Cursor sessions to surface new recurring prompts as draft operons.
type: skill
version: 0.1.0
---

# Operon — Mine Sessions

`operon-setup` is the one-time onboarding skill. `operon-mine` is the
recurring loop: every few weeks (or when the user mentions a new
recurring task), run a mining pass to surface fresh patterns.

## When to use this skill

- The user says "mine my sessions" or "find new patterns" or "what
  has recurred since I last mined."
- It's been more than a couple weeks since their last `operon mine`.
- The user mentions a new repeated workflow they want to capture.

Do not use this for first-run onboarding — use `operon-setup` instead.

## How to use it

### 1. Confirm the user is set up

```
operon doctor --json
```

If `auth.tokenPresent` is false, point them at `operon-setup`. Do not
try to mine without auth — `mine` won't upload drafts to the platform
without a valid token.

### 2. Pick a window

Default mining window is **30 days**. If the user mined recently and
just wants the new stuff, that's right. If they want a broader sweep
(after a sabbatical, after onboarding a new tool, etc.), widen it:

```
operon mine --since 90d
operon mine --since 180d
operon mine --since 365d
```

Wider windows take longer but surface stable patterns that didn't quite
clear the threshold in a 30-day window.

### 3. Run the mine

For interactive review:

```
operon mine --since 30d
```

For agent-driven flow (recommended when an assistant is driving):

```
operon mine --since 30d --yes --json
```

`--yes` accepts every default; `--json` outputs the candidate list as
a single JSON object on stdout.

### 4. Parse the output

The JSON shape is:

```json
{
  "input": 412,
  "kept": 287,
  "clusters": 14,
  "dryRun": false,
  "candidates": [
    {
      "slug": "review-pr",
      "uses": 12,
      "spanDays": 14,
      "score": 0.91,
      "suggestedTools": ["Read", "Bash", "WebFetch"],
      "existing": "new"
    }
  ]
}
```

`existing` tells you how each candidate compares to what's already
published in the workspace:
- `new` — never published
- `same` — already published, no diff
- `changed` — already published, but the draft differs
- `unchecked` — couldn't compare (offline / no workspace)

### 5. Direct the user to review

After mining, drafts auto-upload to
`https://app.withoperon.com/<workspace>/candidates`. From there the
user can edit, polish, and publish each draft.

Or pipe straight into `operon-publish` — call that skill next if the
user wants to ship one of the new drafts immediately.

## Privacy notes

- Mining reads session files in `~/.claude/`, `~/.codex/`, `~/.cursor/`.
  Nothing leaves the laptop without explicit consent.
- After redaction, the candidate metadata (slug, score, uses, prompt
  fingerprint, draft prompt) uploads to the platform's private Drafts
  slot — visible only to the user who mined them.
- Exemplar session **bodies** stay on the laptop. The platform only
  sees opaque session IDs.
- Opt-out: `operon config set candidates.cloudReview false` keeps the
  full mining flow local; no drafts upload.

## Error handling

- **`NO_SOURCES`** → user doesn't have Claude Code / Codex / Cursor
  installed. Tell them to install at least one and retry.
- **`NO_CANDIDATES`** → no patterns crossed the clustering threshold.
  Suggest a wider `--since` window.
- **`RATE_LIMITED`** → wait and retry. The mining itself is local —
  rate limiting hits the draft-upload step. The local on-disk drafts
  under `~/.operon/candidates/` are still authoritative.

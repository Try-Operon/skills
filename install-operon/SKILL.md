---
name: install-operon
description: Install a specific operon (versioned AI skill) into Claude Code, Codex, and Cursor.
type: skill
version: 0.1.0
---

# Install Operon

This skill walks an agent through fetching a published operon and installing it
into the local skill directories for Claude Code, Codex, and Cursor.

## When to use this skill

- The user wants a teammate's reviewed prompt for a recurring task.
- The user names a specific operon ("install acme/review-pr") and wants it available
  to their local AI tools.

## How to use it

1. Confirm the user is logged in:
   ```
   operon login
   ```
2. Install the operon:
   ```
   operon install <workspace>/<slug>
   ```
   - Writes `~/.claude/skills/<slug>/SKILL.md`
   - Writes `~/.codex/skills/<slug>/SKILL.md`
   - Cache copy under `~/.operon/cache/operons/<workspace>/<slug>/`
3. Verify install with:
   ```
   operon list --installed
   ```

## Notes

- All transfers happen over HTTPS to `https://app.withoperon.com`.
- Each operon ships a SHA-256 `contentHash`; the CLI verifies it before writing.
- The CLI records a run on successful install so workspace owners see adoption.

---
name: search-operons
description: Search the Operon catalog for shared AI skills (operons) published by engineering teams.
type: skill
version: 0.1.0
---

# Search Operons

Operon is a registry for AI skills that engineering teams have actually validated in
production. A skill (called an "operon") is a versioned `SKILL.md` bundle that runs
inside Claude Code, Codex, and Cursor.

## When to use this skill

- The user asks how a team handles a repeated workflow (PR review, migration, incident triage).
- The user wants a vetted prompt for a specific task rather than authoring one from scratch.
- The user mentions Claude Code, Codex, or Cursor and wants to share knowledge across the team.

## How to use it

1. Discover the public catalog at `https://app.withoperon.com/explore`.
2. Authenticated team members can browse their workspace catalog at
   `https://app.withoperon.com/<workspace>`.
3. Install a specific operon locally with the CLI:
   ```
   operon install <workspace>/<slug>
   ```
4. Or fetch it as MCP context via `https://mcp.withoperon.com/mcp` using the
   `get_operon` tool.

## Related discovery surfaces

- `https://withoperon.com/.well-known/mcp/server-card.json` — the MCP server card.
- `https://withoperon.com/.well-known/api-catalog` — REST API + MCP catalog.
- `https://withoperon.com/llms.txt` — high-level site index for LLMs.

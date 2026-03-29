# Chirpie Skills for AI Agents

A collection of task-based skills enabling AI coding agents to integrate Chirpie — a social media API for posting to X/Twitter, Bluesky, LinkedIn, and Threads (with X Premium support for long-form posts).

## Directory Structure

```
skills/
├── .claude-plugin/marketplace.json  — Plugin registry
├── chirpie/SKILL.md                 — Router (entry point)
├── chirpie-setup/SKILL.md           — Project setup & quickstart
├── chirpie-posting/SKILL.md         — Posts & threads
├── chirpie-scheduling/SKILL.md      — Scheduled publishing
├── chirpie-sdk/SKILL.md             — TypeScript SDK reference
├── chirpie-cli/SKILL.md             — CLI reference
└── chirpie-mcp/SKILL.md             — MCP server setup
```

## Routing

The `chirpie` router skill directs requests to sub-skills:

| User Intent | Skill |
|-------------|-------|
| "Add Chirpie to my project" | chirpie-setup |
| "Post a tweet" / "Post to Bluesky" / "Post to LinkedIn" / "Post to Threads" / "Create a thread" | chirpie-posting |
| "Schedule a post for tomorrow" | chirpie-scheduling |
| "Use the Chirpie SDK" | chirpie-sdk |
| "Post from the terminal" | chirpie-cli |
| "Set up Chirpie MCP for Claude" | chirpie-mcp |

## Skill Format

Each skill contains a `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: When to activate this skill
---
```

## Key Patterns

- **API Key format:** Always `chirpie_sk_` prefix
- **Base URL:** `https://chirpie.ai/api/v1`
- **Auth header:** `Authorization: Bearer chirpie_sk_YOUR_KEY`
- **Response format:** `{ "data": T }` on success, `{ "error": { "code", "message" } }` on error
- **Environment variable:** `CHIRPIE_API_KEY`
- **Config file:** `~/.chirpie/config.json`

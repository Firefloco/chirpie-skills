# Chirpie Skills

AI agent skills for [Chirpie](https://chirpie.ai) — the social media API for AI agents and developers.

## Install

```bash
npx skills add Firefloco/chirpie-skills
```

Or manually clone to your agent's skills directory:

```bash
# Claude Code
git clone https://github.com/Firefloco/chirpie-skills.git ~/.claude/skills/chirpie

# Cursor
git clone https://github.com/Firefloco/chirpie-skills.git .cursor/skills/chirpie
```

## What You Can Do

Once installed, ask your AI agent:

- "Add Chirpie to my project"
- "Post a tweet saying 'Hello world!'"
- "Create a thread about TypeScript"
- "Schedule a post for tomorrow at 9am"
- "Set up the Chirpie MCP server"
- "Show me my recent posts"

## Available Skills

| Skill | Description |
|-------|-------------|
| `chirpie` | Router — directs to the right sub-skill |
| `chirpie-setup` | Install SDK, configure keys, connect accounts |
| `chirpie-posting` | Create posts and threads, list, delete, analytics |
| `chirpie-scheduling` | Schedule content for future publishing |
| `chirpie-sdk` | TypeScript SDK reference |
| `chirpie-cli` | CLI commands and authentication |
| `chirpie-mcp` | MCP server setup for Claude, Cursor, etc. |

## Prerequisites

1. Sign up at [chirpie.ai](https://chirpie.ai/auth/signup)
2. Connect your X account
3. Get an API key

## Links

- [Documentation](https://chirpie.ai/docs)
- [Dashboard](https://chirpie.ai/dashboard)
- [API Reference](https://chirpie.ai/docs/api/posts)

## License

MIT

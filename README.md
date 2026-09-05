# Chirpie Skills

AI agent skills for [Chirpie](https://chirpie.ai) — the social media API for AI agents and developers. Post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram from a single API.

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
- "Post to Bluesky saying 'Hello world!'"
- "Post to LinkedIn saying 'Just shipped a new feature!'"
- "Post to Threads saying 'Hello world!'"
- "Post to Mastodon saying 'Hello fediverse!'"
- "Post to Instagram with this photo"
- "Post to our Facebook Page about the launch"
- "Send a message to our Telegram channel"
- "Post to Reddit in r/programming"
- "Pin this image to my Pinterest board"
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
| `chirpie-openclaw` | OpenClaw setup — skill install, MCP wiring, agent guardrails |

## OpenClaw

```bash
openclaw skills install git:Firefloco/chirpie-skills
openclaw skills install git:Firefloco/chirpie-skills --global
```

Set `CHIRPIE_API_KEY` in the agent's environment. See the [OpenClaw guide](https://chirpie.ai/docs/guides/give-your-openclaw-agent-a-social-media-account).

## Other Ways to Use Chirpie

- **n8n** — install the `@chirpie/n8n-nodes-chirpie` community node (Settings → Community Nodes → `@chirpie/n8n-nodes-chirpie`) to post, thread, schedule, and pull analytics from a workflow. It also works as a tool for n8n AI Agents. [Docs](https://chirpie.ai/docs/n8n)

## Prerequisites

1. Sign up at [chirpie.ai](https://chirpie.ai/auth/signup)
2. Connect your social accounts (X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, Telegram, Reddit, Pinterest, TikTok, YouTube, Google Business Profile, and/or Snapchat)
3. Get an API key

## Links

- [Documentation](https://chirpie.ai/docs)
- [Dashboard](https://chirpie.ai/dashboard)
- [API Reference](https://chirpie.ai/docs/api/posts)

## License

MIT

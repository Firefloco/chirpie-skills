# Chirpie Skills

AI agent skills for [Chirpie (chirpie.ai)](https://chirpie.ai?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme), the publishing connector for AI agents. Post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram from a single API.

## Install

### Claude Code (recommended)

Install the Chirpie plugin. It bundles the hosted MCP server **and** all eight skills, so Claude gets the Chirpie tools and the know-how in one step:

```
/plugin marketplace add Firefloco/chirpie-skills
/plugin install chirpie@chirpie-skills
```

The plugin registers `https://chirpie.ai/mcp` and signs you in with OAuth the first time a Chirpie tool runs. There is no API key to paste.

From a shell, the same two steps are `claude plugin marketplace add Firefloco/chirpie-skills` and `claude plugin install chirpie@chirpie-skills`. Then run `/mcp` and sign in to **chirpie**.

### Any other agent

Connect the hosted MCP server, `https://chirpie.ai/mcp`, in your agent (one click in Cursor and VS Code, one command in Codex and Gemini CLI: see the [per-app guides](https://chirpie.ai/docs/mcp?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme)), then add the skills:

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
- "Create a thread about TypeScript"
- "Schedule a post for tomorrow at 9am"
- "Set up the Chirpie MCP server"
- "Show me my recent posts"

## Available Skills

| Skill | Description |
|-------|-------------|
| `chirpie` | Router: directs to the right sub-skill |
| `chirpie-setup` | Install SDK, configure keys, connect accounts |
| `chirpie-posting` | Create posts and threads, list, delete, analytics |
| `chirpie-scheduling` | Schedule content for future publishing |
| `chirpie-sdk` | TypeScript SDK reference |
| `chirpie-cli` | CLI commands and authentication |
| `chirpie-mcp` | MCP server setup for Claude, Cursor, etc. |
| `chirpie-openclaw` | OpenClaw setup: skill install, MCP wiring, agent guardrails |

## OpenClaw

```bash
openclaw skills install git:Firefloco/chirpie-skills
openclaw skills install git:Firefloco/chirpie-skills --global
```

Set `CHIRPIE_API_KEY` in the agent's environment. See the [OpenClaw guide](https://chirpie.ai/docs/guides/give-your-openclaw-agent-a-social-media-account?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme).

## Other Ways to Use Chirpie

- **n8n** (self-hosted): install the `@chirpie/n8n-nodes-chirpie` community node (Settings → Community Nodes → `@chirpie/n8n-nodes-chirpie`) to post, thread, schedule, and pull analytics from a workflow. It also works as a tool for n8n AI Agents. n8n Cloud can only install verified community nodes, and this one is not verified yet. [Docs](https://chirpie.ai/docs/n8n?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme)

## Prerequisites

1. A Chirpie account. Signing in from your AI app the first time creates one, or sign up at [chirpie.ai](https://chirpie.ai/auth/signup?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme)
2. Your social accounts connected (X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, Telegram)
3. An API key only if your agent calls the CLI, the SDK or the REST API, or uses an MCP client that cannot sign in (such as Manus)

## Links

- [Documentation](https://chirpie.ai/docs?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme)
- [Dashboard](https://chirpie.ai/dashboard?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme)
- [API Reference](https://chirpie.ai/docs/api/posts?utm_source=agent_skills&utm_medium=skill&utm_campaign=readme)

## License

MIT

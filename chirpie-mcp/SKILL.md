---
name: chirpie-mcp
description: Connect the Chirpie MCP server to Claude, Claude Code, Cursor, ChatGPT, or other AI agents — hosted (one URL) or local. Lists all available tools and their parameters.
---

# Chirpie MCP Server

The Chirpie MCP server lets AI agents post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, Telegram, Reddit, Pinterest, TikTok, YouTube, Google Business Profile, and Snapchat (14 platforms) through the Model Context Protocol.

There are two ways to connect. Prefer the hosted server unless the user explicitly wants to run it locally.

## Hosted server (recommended)

One URL, nothing to install:

```
https://chirpie.ai/mcp
```

The user signs in through their client's connector flow the first time. No API key needed.

### Claude Code

```bash
claude mcp add --transport http chirpie https://chirpie.ai/mcp
```

Then `/mcp` → select **chirpie** → sign in.

Or add to `.mcp.json` in the project:

```json
{
  "mcpServers": {
    "chirpie": {
      "type": "http",
      "url": "https://chirpie.ai/mcp"
    }
  }
}
```

### Claude (claude.ai / desktop)

Settings → Connectors → **Add custom connector** → name `Chirpie`, URL `https://chirpie.ai/mcp` → Connect.

### Cursor

Add to `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global):

```json
{
  "mcpServers": {
    "chirpie": {
      "url": "https://chirpie.ai/mcp"
    }
  }
}
```

### ChatGPT

Settings → Connectors → Create → **MCP server** → URL `https://chirpie.ai/mcp` → OAuth.

### With an API key instead of signing in

For CI, scripts, or clients without an OAuth flow:

```json
{
  "mcpServers": {
    "chirpie": {
      "type": "http",
      "url": "https://chirpie.ai/mcp",
      "headers": { "Authorization": "Bearer chirpie_sk_..." }
    }
  }
}
```

## Local server (alternative)

Authenticate via the CLI (opens browser — user is typically already logged in):

```bash
npm install -g chirpie
chirpie login
```

The local server reads saved credentials from `~/.chirpie/config.json` automatically. Alternatively, set `CHIRPIE_API_KEY` as an environment variable (useful for CI/CD).

### Claude Code

```bash
claude mcp add chirpie -- npx @chirpie/mcp
```

Or add to `.mcp.json` in your project:

```json
{
  "mcpServers": {
    "chirpie": {
      "type": "stdio",
      "command": "npx",
      "args": ["@chirpie/mcp"]
    }
  }
}
```

### Claude Desktop

Add to `~/Library/Application Support/Claude/claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "chirpie": {
      "command": "npx",
      "args": ["@chirpie/mcp"]
    }
  }
}
```

### Cursor

Add to Cursor MCP settings:

```json
{
  "mcpServers": {
    "chirpie": {
      "command": "npx",
      "args": ["@chirpie/mcp"]
    }
  }
}
```

Both servers expose exactly the same tools.

## Available Tools

### chirpie_post

Create a single post on any of 14 platforms with optional media. Note: Instagram, Pinterest, TikTok, YouTube, and Snapchat REQUIRE media. Facebook is Pages only.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID (any of 14 platforms) |
| `text` | string | Yes | Post text. Max varies: X 280 (25K Premium), Bluesky 300, LinkedIn 3K, Threads 500, Mastodon 500, Instagram 2,200, Facebook 63,206, Telegram 4,096, Reddit 40K, Pinterest 500, TikTok 2,200, YouTube 5K, Google Business 1,500, Snapchat 160. |
| `media_urls` | string[] | No | Public image/video URLs. Limits vary by platform. Instagram, Pinterest, TikTok, YouTube, and Snapchat REQUIRE media. |
| `schedule_at` | string | No | ISO 8601 datetime (UTC, must be future) |

### chirpie_thread

Create a multi-post thread on any of 14 platforms. X, Bluesky, Threads, Mastodon, and Telegram support native threading. Reddit threads via comments. Others degrade to standalone posts.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID (any of 14 platforms) |
| `posts` | array | Yes | Array of `{ text, media_urls? }` objects (2-25). Media limits vary by platform. |
| `schedule_at` | string | No | ISO 8601 datetime |

### chirpie_list_posts

List posts with optional filters.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter: draft, scheduled, published, failed, deleted |
| `account_id` | string | No | Filter by account |
| `limit` | number | No | Results to return |

### chirpie_get_post

Get a single post by ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID |

### chirpie_delete_post

Delete a post (also removes from platform if published, except TikTok which has no delete API).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID |

### chirpie_list_accounts

List all connected accounts (all 14 platforms). No parameters.

Returns: `id`, `platform`, `username`, `display_name`, `avatar_url`, `is_active`

### chirpie_analytics

Get engagement metrics for a published post.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `post_id` | string | Yes | Post UUID |

Returns: impressions, likes, retweets, replies, quotes, bookmarks, clicks.

## Example Prompts

Once configured, ask your AI agent:

- "Post a tweet saying 'Just shipped v2!'"
- "Post to Bluesky saying 'Just shipped v2!'"
- "Post to LinkedIn saying 'Just shipped v2!'"
- "Post to Threads saying 'Just shipped v2!'"
- "Post to Mastodon saying 'Hello fediverse!'"
- "Post to Instagram with this image" (requires media)
- "Post to our Facebook Page about the product launch"
- "Send a message to our Telegram channel"
- "Post to Reddit in r/programming"
- "Pin this image to my Pinterest board"
- "Create a thread about why TypeScript is great"
- "Show me my recent posts"
- "What are the analytics for my last published post?"
- "Schedule a post for tomorrow at 9am UTC"
- "List my connected accounts"
- "Delete the post with ID xyz"

## Connecting social accounts from chat

The user does not need to leave the agent to connect a platform. Call the matching
`chirpie_connect_*` tool and hand back the `authorization_url` for them to open:

| Tool | Platform | Arguments |
|------|----------|-----------|
| `chirpie_connect_x` | X/Twitter | none |
| `chirpie_connect_linkedin` | LinkedIn | none |
| `chirpie_connect_threads` | Threads | none |
| `chirpie_connect_instagram` | Instagram | none |
| `chirpie_connect_facebook` | Facebook Pages | none |
| `chirpie_connect_bluesky` | Bluesky | `identifier`, `app_password` |
| `chirpie_connect_mastodon` | Mastodon | `instance_url` |
| `chirpie_connect_telegram` | Telegram | `bot_token`, `chat_id` |

## Key management tools

`chirpie_create_key` (returns the key once), `chirpie_list_keys`, `chirpie_revoke_key`.

## Authentication Priority

**Hosted server** — either:
1. Sign in through the client's connector flow (recommended)
2. `Authorization: Bearer chirpie_sk_...` header

**Local server**:
1. `CHIRPIE_API_KEY` environment variable
2. `~/.chirpie/config.json` (created by `chirpie login`)

Both CLI and the local MCP server share the same config — `chirpie login` authenticates both.

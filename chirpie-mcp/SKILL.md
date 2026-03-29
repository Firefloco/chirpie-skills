---
name: chirpie-mcp
description: Connect the Chirpie MCP server to Claude Code, Claude Desktop, Cursor, or other AI agents. Lists all available tools and their parameters.
---

# Chirpie MCP Server

The Chirpie MCP server lets AI agents post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, Telegram, Reddit, Pinterest, TikTok, YouTube, Google Business Profile, and Snapchat (14 platforms) through the Model Context Protocol.

## Prerequisites

Authenticate via the CLI (opens browser — user is typically already logged in):

```bash
npm install -g chirpie
chirpie login
```

The MCP server reads saved credentials from `~/.chirpie/config.json` automatically. Alternatively, set `CHIRPIE_API_KEY` as an environment variable (useful for CI/CD).

## Setup

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

## Authentication Priority

1. `CHIRPIE_API_KEY` environment variable
2. `~/.chirpie/config.json` (created by `chirpie login`)

Both CLI and MCP share the same config — `chirpie login` authenticates both.

---
name: chirpie-mcp
description: Connect the Chirpie MCP server to Claude Code, Claude Desktop, Cursor, or other AI agents. Lists all available tools and their parameters.
---

# Chirpie MCP Server

The Chirpie MCP server lets AI agents post to X/Twitter, Bluesky, LinkedIn, and Threads through the Model Context Protocol.

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

Create a single post on X, Bluesky, LinkedIn, or Threads with optional media (images/video).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID (X, Bluesky, LinkedIn, or Threads) |
| `text` | string | Yes | Post text. X: max 280 chars (25,000 for X Premium). Bluesky: max 300 chars. LinkedIn: max 3,000 chars. Threads: max 500 chars. |
| `media_urls` | string[] | No | Up to 4 public image/video URLs (JPEG, PNG, WebP, GIF up to 5MB; MP4/MOV up to 512MB). Bluesky: images only (~1MB each, max 4), no video. LinkedIn: images only (JPEG/PNG/GIF, max 8MB each, max 4), no video. Threads: one image per post (URL-based, JPEG/PNG), no video. |
| `schedule_at` | string | No | ISO 8601 datetime (UTC, must be future) |

### chirpie_thread

Create a multi-post thread on X, Bluesky, LinkedIn, or Threads.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID (X, Bluesky, LinkedIn, or Threads) |
| `posts` | array | Yes | Array of `{ text, media_urls? }` objects (2-25). Bluesky: images only (~1MB each, max 4), no video. LinkedIn: images only (JPEG/PNG/GIF, max 8MB each, max 4), no video. Threads: one image per post (URL-based, JPEG/PNG), no video. |
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

Delete a post (also removes from X/Bluesky/LinkedIn/Threads if published).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID |

### chirpie_list_accounts

List all connected accounts (X, Bluesky, LinkedIn, and Threads). No parameters.

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

---
name: chirpie-mcp
description: Connect the Chirpie MCP server to Claude, Claude Code, Cursor, ChatGPT, or other AI agents, hosted (one URL) or local. Lists all available tools and their parameters.
---

# Chirpie MCP Server

The Chirpie MCP server lets AI agents post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram through the Model Context Protocol.

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

Authenticate via the CLI (opens a browser; the user is typically already logged in):

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

### chirpie_upload_media

Upload an image or a video and get back the id a post can attach. Use it whenever the file is not already on a public URL.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string | No | A public image or video URL for Chirpie to fetch and store |
| `data` | string | No | The file's bytes, base64 encoded. Use this or `url`, not both |
| `filename` | string | No | A name to show in the dashboard. Never used to decide the file type |

The file type is read from the file's own first bytes, so a wrong extension does not matter and a mislabelled file is refused. The id is valid for 7 days; attach it with `media: [{ "id": "...", "alt": "..." }]`. Uploads are limited to 3 MB of file when sent as `data`, so a larger file goes in `media_urls` instead, which has no such limit.

### chirpie_post

Create a single post on any connected platform with optional media. Note: Instagram REQUIRES media. Facebook is Pages only.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID |
| `text` | string | Yes | Post text. Max varies: X 280 (25,000 on Premium), Bluesky 300, LinkedIn 3,000, Threads 500, Mastodon 500, Instagram 2,200, Facebook 63,206, Telegram 4,096. |
| `media` | object[] | No | Uploaded files and public links, each `{ id? , url?, alt? }`, with **either** `id` (from `chirpie_upload_media`) **or** `url` per item, never both. `alt` describes the item for screen readers. Use this **or** `media_urls`, not both. |
| `media_urls` | string[] | No | Public image/video URLs, for a post that needs no alt text. Max per post: X 4, Bluesky 4, LinkedIn 4, Threads 1, Mastodon 4, Instagram 10, Facebook 10, Telegram 10. Instagram REQUIRES media. |
| `schedule_at` | string | No | ISO 8601 datetime, must be future and carry a timezone (`...Z` or `+02:00`); normalized to UTC |

### chirpie_thread

Create a multi-post thread on any connected platform. X, Bluesky, Threads, Mastodon, and Telegram support native threading. Others degrade to standalone posts.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID |
| `posts` | array | Yes | Array of `{ text, media?, media_urls? }` objects (2-25). Media limits vary by platform. |
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

### chirpie_update_post

Edit a post that has not published yet. Leaving `schedule_at` out keeps the time the post already has, so this never publishes anything. Only a post that has not published yet can be edited: one that is publishing, published, failed or deleted answers `409 post_not_editable`. Editing does not count against the monthly quota. Rescheduling one post of a thread moves every part of it, and the response lists them in `rescheduled_post_ids`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID |
| `text` | string | No | Replacement text |
| `media` | object[] | No | Replacement media, each `{ id? , url?, alt? }`. An empty array removes the media |
| `media_urls` | string[] | No | Replacement media URLs. An empty array removes the media |
| `schedule_at` | string | No | New ISO 8601 publish time, in the future and carrying a timezone |

### chirpie_delete_post

Delete a post (also removes it from the platform if published, except Instagram, which has no delete API). Deleting any post of a scheduled thread cancels the whole thread.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID |

### chirpie_list_accounts

List all connected accounts. No parameters.

Returns: `id`, `platform`, `username`, `display_name`, `avatar_url`, `is_active`, and `inactive_reason`. Pause on `is_active: false`, then read the reason: `plan_limit` means switch it on with `chirpie_activate_account` once a slot is free, `token_revoked` / `token_expired` / `reauth_required` mean it has to be connected again, and **no reason at all** means the customer switched it off deliberately, so `chirpie_activate_account` is the fix there too.
X accounts also return `byo_keys`, which is true when the account posts through the
user's own X developer app. Inactive accounts are included; `inactive_reason:
"plan_limit"` means the account is connected but was not switched on because the
plan's account limit was already full (one Facebook authorization can grant
several Pages, and all of them are stored rather than dropped). LinkedIn accounts
also return `account_type`: `"member"` for the user's own profile,
`"organization"` for a LinkedIn Page they administer. Both publish the same way.

### chirpie_activate_account

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID |

Switch an account on so it can publish. Fails with a plan-limit error when no
slot is free, so deactivate another account first.

### chirpie_deactivate_account

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID |

Switch an account off. It stays connected and frees a slot against the plan's
account limit, so another one can be activated without reauthorizing.

**This cancels the account's scheduled posts.** A deactivated account cannot
publish, so every post queued against it is cancelled and its monthly quota
returned; `canceled_posts` in the response says how many. Activating the account
again does not restore them. Read `scheduled_posts` from `chirpie_list_accounts`
first and confirm with the user before deactivating an account that has any.

### chirpie_disconnect_account

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes | Account UUID |

End the connection. The account stops publishing straight away and stops counting
against the plan's account limit, and the credential Chirpie stored for it is
removed, so connecting it again means authorizing it on the platform again.

**This cancels the account's scheduled posts too.** Every post queued against it is
cancelled and its monthly quota returned; `canceled_posts` in the response says how
many, and they are not restored. Posts the account already published are kept.

This cannot be undone, so always confirm with the user first. Use
`chirpie_deactivate_account` instead when the account should come back later
without reauthorizing.

### chirpie_set_x_keys / chirpie_get_x_keys_status / chirpie_remove_x_keys

Manage the user's own X developer app, so their X accounts post against their X
API credits and X link posts are not surcharged.
Setup guide: https://chirpie.ai/docs/x-byo-keys

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `client_id` | string | Yes (set) | OAuth 2.0 Client ID from developer.x.com |
| `client_secret` | string | Yes (set) | OAuth 2.0 Client Secret. Stored encrypted, never returned |
| `label` | string | No | A name for the app |

`chirpie_get_x_keys_status` and `chirpie_remove_x_keys` take no parameters.
After setting keys, the user must add the returned `redirect_uri` as a callback
URI on their X app and reconnect each X account (`chirpie_connect_x`).

`chirpie_remove_x_keys` is API-key auth only. On a hosted server signed in with
OAuth it is not offered. Remove the app from
[account settings](https://chirpie.ai/dashboard/accounts) instead. Setting keys
and checking their status stay available over OAuth.

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
| `chirpie_connect_linkedin` | LinkedIn profile | none |
| `chirpie_connect_linkedin_pages` | LinkedIn Pages (coming soon) | none |
| `chirpie_connect_threads` | Threads (coming soon) | none |
| `chirpie_connect_instagram` | Instagram (coming soon) | none |
| `chirpie_connect_facebook` | Facebook Pages (coming soon) | none |
| `chirpie_connect_bluesky` | Bluesky | `identifier`, `app_password` |
| `chirpie_connect_mastodon` | Mastodon | `instance_url` |
| `chirpie_connect_telegram` | Telegram | `bot_token`, `chat_id` |

## Key management tools

`chirpie_create_key` (returns the key once), `chirpie_list_keys`, `chirpie_revoke_key`.

These are API-key auth only, along with `chirpie_remove_x_keys`. On a hosted
server signed in with OAuth they are not offered. Manage API keys in the
[dashboard](https://chirpie.ai/dashboard/keys) instead.

## Authentication Priority

**Hosted server** (either):
1. Sign in through the client's connector flow (recommended)
2. `Authorization: Bearer chirpie_sk_...` header

**Local server**:
1. `CHIRPIE_API_KEY` environment variable
2. `~/.chirpie/config.json` (created by `chirpie login`)

Both CLI and the local MCP server share the same config, so `chirpie login` authenticates both.

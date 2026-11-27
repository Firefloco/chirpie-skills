---
name: chirpie-mcp
description: Connect Chirpie (chirpie.ai) to Claude, Claude Code, Cursor, VS Code, ChatGPT, Codex, Windsurf, Gemini CLI, Manus, Meta Muse, n8n or any MCP client through the hosted server at https://chirpie.ai/mcp, or run it locally. Lists all available tools and their parameters, including first comments and saving and promoting drafts.
---

# Chirpie MCP Server

The Chirpie MCP server lets AI agents post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram through the Model Context Protocol.

Always set up the hosted server, `https://chirpie.ai/mcp`, unless the user explicitly needs a local stdio server (a stdio-only client, or files read straight from their disk). The hosted server is always current, needs no install and no API key, and takes large files through `chirpie_create_media_upload`.

## Hosted server (recommended)

One URL, nothing to install:

```
https://chirpie.ai/mcp
```

The user signs in through their client's connector flow the first time. No API key needed.

**The sign-in is the user's to do, not yours.** When you add the server for them, tell them what comes next, in these words or close to them: "A browser window will open on chirpie.ai. Sign in, or choose Sign up and enter the code Chirpie emails you. Then choose Full access or Read only and click Allow, and tell me when the page says you are connected." In Claude Code, the user runs `/mcp` and picks **chirpie** themselves; it is a slash command, not something you can run from a tool. Never ask for their Chirpie password or the emailed code in the chat.

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

Quickest: add Chirpie from its Claude Connectors Directory listing, https://claude.ai/directory/chirpie, and sign in. Or by URL: Customize → Connectors (Claude Desktop: Settings → Connectors) → **+** → **Add custom connector** → URL `https://chirpie.ai/mcp` → Add → Connect. Full guide: https://chirpie.ai/docs/mcp/claude?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp

### Meta Muse

In a Muse conversation: "Add a custom connector for Chirpie. It is a remote MCP server over Streamable HTTP at https://chirpie.ai/mcp with OAuth sign-in. Send me the sign-in link and I will approve it." Full guide: https://chirpie.ai/docs/mcp/meta-muse?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp

### Cursor

One click: open `cursor://anysphere.cursor-deeplink/mcp/install?name=chirpie&config=eyJ1cmwiOiJodHRwczovL2NoaXJwaWUuYWkvbWNwIn0=` (web version: `https://cursor.com/install-mcp?name=chirpie&config=eyJ1cmwiOiJodHRwczovL2NoaXJwaWUuYWkvbWNwIn0%3D`). Or add to `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global):

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

On the web only. **Plugins** → **+** → **Add custom MCP server** → name `Chirpie`, URL `https://chirpie.ai/mcp` → OAuth → **Create as a plugin** (sign in to Chirpie). No developer-mode toggle is needed first. If the workspace still shows the older flow, turn on Developer mode under Settings → Apps → Advanced settings and create the app there. In a chat, pick Chirpie from the tools menu or mention it with `@`. Publishing needs full MCP support (Business, Enterprise, Education); Pro gets read and fetch only, and on Business only admins and owners can add it, so tell a member to ask their admin. ChatGPT treats tools not marked read-only as writes and may ask before running them. For the OpenAI Responses API, add `{ "type": "mcp", "server_label": "chirpie", "server_url": "https://chirpie.ai/mcp", "authorization": "<chirpie_sk_ key>" }` to `tools`. Full guide: https://chirpie.ai/docs/mcp/chatgpt?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp

### VS Code (GitHub Copilot)

One click: open `vscode:mcp/install?%7B%22name%22%3A%22chirpie%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Fchirpie.ai%2Fmcp%22%7D`, or run:

```bash
code --add-mcp '{"name":"chirpie","type":"http","url":"https://chirpie.ai/mcp"}'
```

Workspace file `.vscode/mcp.json` (top-level key `servers`): `{ "servers": { "chirpie": { "type": "http", "url": "https://chirpie.ai/mcp" } } }`. VS Code opens a browser to sign in on first start.

### Codex

```bash
codex mcp add chirpie --url https://chirpie.ai/mcp
codex mcp login chirpie
```

Or `~/.codex/config.toml`: `[mcp_servers.chirpie]` with `url = "https://chirpie.ai/mcp"`.

### Gemini CLI

```bash
gemini mcp add --transport http chirpie https://chirpie.ai/mcp
```

Then `/mcp auth chirpie` inside Gemini CLI. In `settings.json` the key is `httpUrl`, not `url` (which means SSE).

### Windsurf

Open the MCP config file from Cascade's MCP settings and add `{ "mcpServers": { "chirpie": { "serverUrl": "https://chirpie.ai/mcp" } } }`, then refresh and sign in.

### Manus

Manus's Add MCP by URL form has no field for a key, so use **Connectors** → **Create** → **Import MCP by JSON** with `{ "mcpServers": { "chirpie": { "url": "https://chirpie.ai/mcp", "headers": { "Authorization": "Bearer chirpie_sk_..." } } } }`. Ask the user to create a scoped key in the dashboard (https://chirpie.ai/dashboard/keys) and paste it into Manus themselves, not into the chat.

### n8n

Give an AI Agent an **MCP Client Tool** with endpoint `https://chirpie.ai/mcp`, HTTP Streamable transport, and an MCP OAuth2 credential. OAuth needs n8n served over https or on localhost; a self-hosted n8n on plain http elsewhere should use a scoped API key as a bearer header.

Per-app guides: https://chirpie.ai/docs/mcp?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp

### With an API key instead of signing in

For CI, scripts, or clients without an OAuth flow. This is the Claude Code (`.mcp.json`) and Cursor (`mcp.json`) shape:

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

Other clients take the same `headers` object inside their own entry: VS Code `{ "servers": { "chirpie": { "type": "http", "url": "https://chirpie.ai/mcp", "headers": {...} } } }`, Windsurf beside `serverUrl`, Gemini CLI beside `httpUrl`. Codex: `bearer_token_env_var = "CHIRPIE_API_KEY"` under `[mcp_servers.chirpie]`, with the key in that variable.

## Advanced: local server

Only when the hosted server will not do: a client that speaks nothing but stdio, or an agent that should read large files straight from the user's disk with `path`.

Authenticate via the CLI (opens a browser; the user is typically already logged in):

```bash
npm install -g chirpie
chirpie login
```

The local server reads saved credentials from `~/.chirpie/config.json` automatically. Alternatively, set `CHIRPIE_API_KEY` as an environment variable (useful for CI/CD).

### Claude Code

```bash
claude mcp add chirpie -- npx -y @chirpie/mcp
```

Or add to `.mcp.json` in your project:

```json
{
  "mcpServers": {
    "chirpie": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@chirpie/mcp"]
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
      "args": ["-y", "@chirpie/mcp"]
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
      "args": ["-y", "@chirpie/mcp"]
    }
  }
}
```

Both servers expose exactly the same tools.

### Tool annotations

Every tool carries a title and the MCP annotations `readOnlyHint`,
`destructiveHint`, `idempotentHint` and `openWorldHint`, and clients use them to
decide when to ask the user first (by default, Claude asks before a destructive tool).

- **Read-only**: `chirpie_list_posts`, `chirpie_get_post`, `chirpie_list_accounts`, `chirpie_analytics`, `chirpie_list_comments`, `chirpie_list_keys`, `chirpie_get_x_keys_status`.
- **Destructive**: `chirpie_delete_post`, `chirpie_delete_comment`, `chirpie_update_post` (overwrites a queued post), `chirpie_deactivate_account` and `chirpie_disconnect_account` (both cancel the account's scheduled posts), `chirpie_revoke_key`, `chirpie_set_x_keys` (replaces any app already set), `chirpie_remove_x_keys`.
- **Neither**: everything that creates (posts, threads, media, replies, keys, connects) and the reversible toggles (`chirpie_hide_post`, `chirpie_unhide_post`, `chirpie_hide_comment`, `chirpie_activate_account`).
- No creating tool is marked idempotent. `chirpie_post`, `chirpie_thread`, `chirpie_upload_media`, `chirpie_create_media_upload`, `chirpie_reply_to_comment` and `chirpie_retry_first_comment` take an `idempotency_key` that makes a retry safe; the other creating tools have none.

## Available Tools

### chirpie_upload_media

Upload an image or a video and get back the id a post can attach. Use it whenever the file is not already on a public URL.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `path` | string | No | **Local (stdio) server only.** A file on this computer, at any supported size. Files over 4 MB go straight to storage |
| `url` | string | No | A public image or video URL for Chirpie to fetch and store |
| `data` | string | No | The file's bytes, base64 encoded, up to 3 MB of file |
| `upload_id` | string | No | Finishes a direct upload started with `chirpie_create_media_upload`, once the file has been sent |
| `filename` | string | No | A name to show in the dashboard. Never used to decide the file type |
| `idempotency_key` | string | No | Makes retrying this call safe. The same key with the same request replays the first answer for 24 hours instead of sending it again; the same key with a different request is `422 idempotency_key_reused`; a retry arriving while the first is still running is `409 idempotency_in_progress`, which does not wait, so retry once more to collect the replay. Max 255 characters |

Give it one of `path`, `url`, `data` or `upload_id`. The file type is read from the file's own first bytes, so a wrong extension does not matter and a mislabelled file is refused. The id is valid for 7 days; attach it with `media: [{ "id": "...", "alt": "..." }]`.

**A large local file.** On the local server, pass its `path`: up to 20 MB for an image and 512 MB for a video, no bucket or public URL needed. The hosted server (`https://chirpie.ai/mcp`) cannot read your computer's files and does not offer `path`; there, call `chirpie_create_media_upload`, send the file to the `upload_url` it returns, then call this with the `upload_id`.

### chirpie_create_media_upload

Start a direct upload for a file too large to send as base64 `data`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content_type` | string | Yes | `image/jpeg`, `image/png`, `image/webp`, `image/gif`, `video/mp4` or `video/quicktime` |
| `size` | integer | Yes | The file's exact size in bytes. Up to 20 MB for an image, 512 MB for a video |
| `filename` | string | No | A name to show in the dashboard |
| `idempotency_key` | string | No | The same key with the same request hands back the same target. The same key can then be passed to `chirpie_upload_media` with the `upload_id`: the two calls never collide |

Returns `upload_id`, `upload_url`, `method` and `headers`. Send the file's bytes as the body to `upload_url` with exactly that method and those headers, for example `curl -X PUT --data-binary @video.mp4 -H "<name>: <value>" ... "<upload_url>"` (no API key: the target carries its own authorization), then call `chirpie_upload_media` with the `upload_id`. The target closes after an hour.

### chirpie_post

Create a single post on any connected platform with optional media, on one account or on several at once. Note: Instagram REQUIRES media. Facebook is Pages only.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes, unless `account_ids` is given | Account UUID |
| `account_ids` | string[] | Yes, unless `account_id` is given | 1 to 25 account UUIDs. Publishes to all of them in one call and answers with a `group_id` plus one result per account, in the order given |
| `account_configurations` | object | No | Per-account overrides keyed by account UUID, each taking `text`, `first_comment` (`""` publishes that account with none), `configuration` (replacing the whole block for that account) and media. Only with `account_ids`, and every key must be in it. A field left out inherits the call's own; media replaces rather than merges, and `media: []` publishes that account with none |
| `text` | string | Yes, unless `draft` is true | Post text. Max varies: X 280 (25,000 on Premium), Bluesky 300, LinkedIn 3,000, Threads 500, Mastodon 500, Instagram 2,200, Facebook 63,206, Telegram 4,096. |
| `media` | object[] | No | Uploaded files and public links, each `{ id? , url?, alt? }`, with **either** `id` (from `chirpie_upload_media`) **or** `url` per item, never both. `alt` describes the item for screen readers. Use this **or** `media_urls`, not both. |
| `media_urls` | string[] | No | Public image/video URLs, for a post that needs no alt text. Max per post: X 4, Bluesky 4, LinkedIn 4, Threads 20, Mastodon 4, Instagram 10, Facebook 10, Telegram 10. Video on X, Bluesky (MP4, 100 MB), LinkedIn (MP4, 500 MB), Threads, Mastodon, Instagram, Facebook and Telegram (inside a carousel too on Threads and Instagram). Instagram REQUIRES media. |
| `first_comment` | string | No | A comment published under the post the moment it goes out. X, Threads, Instagram and Facebook only: anywhere else the call is refused with `400 first_comment_unsupported` rather than the comment dropped. Counts as one post against the quota. See "The first comment" below |
| `configuration` | object | No | Per-platform publishing options keyed by platform. Instagram takes `feed` (the default), `story` or `reel`; a Facebook Page takes `feed` or `story`. An option the platform or the placement does not carry is refused with `400 configuration_unsupported`, never dropped. See "Publishing options" below |
| `schedule_at` | string | No | ISO 8601 datetime, must be future. Either absolute, carrying a timezone (`...Z` or `+02:00`), normalized to UTC, or a local time with no offset (`2026-11-01T09:30:00`) read in `timezone` or the timezone saved on the account. A local time with neither is refused. On a draft it is only the time to remember |
| `timezone` | string | No | The IANA zone a `schedule_at` with no offset is read in, such as `America/New_York`. Daylight saving is resolved for the date named, which is what a client computing today's offset gets wrong across a clock change. Ignored when `schedule_at` already carries an offset. Leave it out to use the timezone saved on the account. A fixed offset like `+02:00` is NOT accepted here |
| `idempotency_key` | string | No | Makes retrying this call safe. The same key with the same request replays the first answer for 24 hours instead of sending it again; the same key with a different request is `422 idempotency_key_reused`; a retry arriving while the first is still running is `409 idempotency_in_progress`, which does not wait, so retry once more to collect the replay. Max 255 characters |
| `draft` | boolean | No | Save the post without sending it. Nothing reaches the platform and nothing counts against the quota. The answer carries `warnings`: see "Drafts" below |
| `allow_duplicate` | boolean | No | Publish even when an account already has a post with the same text within 12 hours. Without it such a call is refused with `409 duplicate_post`, nothing is sent, and `duplicate_of` names the post already there |

### chirpie_thread

Create a multi-post thread on any connected platform, on one account or on several at once. X, Bluesky, Threads, Mastodon, and Telegram support native threading. Others degrade to standalone posts.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `account_id` | string | Yes, unless `account_ids` is given | Account UUID |
| `account_ids` | string[] | Yes, unless `account_id` is given | 1 to 25 account UUIDs. Publishes the thread to all of them in one call and answers with a `group_id` plus one result per account |
| `account_configurations` | object | No | Per-account overrides keyed by account UUID, each taking `posts`, which replaces the whole array for that account (still 2-25 parts, or 1-25 on a draft), `first_comment` (`""` publishes that account with none) and `configuration`. Only with `account_ids`, and every key must be one of the ids named there |
| `posts` | array | Yes | Array of `{ text, media?, media_ids?, media_urls? }` objects (2-25, or 1-25 on a draft). Media limits vary by platform. |
| `first_comment` | string | No | One comment for the whole thread, published under the **last** part and reported on that part |
| `configuration` | object | No | Per-platform publishing options, applying to every part alike. A thread publishes to the feed, so an Instagram story or reel, and a Facebook Page story, are refused: each is a single post |
| `schedule_at` | string | No | ISO 8601 datetime, applying to the whole group. Either absolute, carrying a timezone, or a local time with no offset read in `timezone` or the timezone saved on the account. On a draft it is only the time to remember |
| `timezone` | string | No | The IANA zone a `schedule_at` with no offset is read in, such as `America/New_York`. Daylight saving is resolved for the date named, which is what a client computing today's offset gets wrong across a clock change. Ignored when `schedule_at` already carries an offset. Leave it out to use the timezone saved on the account. A fixed offset like `+02:00` is NOT accepted here |
| `idempotency_key` | string | No | Makes retrying this call safe. The same key with the same request replays the first answer for 24 hours instead of sending it again; the same key with a different request is `422 idempotency_key_reused`; a retry arriving while the first is still running is `409 idempotency_in_progress`, which does not wait, so retry once more to collect the replay. Max 255 characters |
| `draft` | boolean | No | Save the thread without sending it. A draft thread may be a single part while it is still being written |
| `allow_duplicate` | boolean | No | Publish even when an account already has a post with the same text as the thread's first post within 12 hours |

**Reading a multi-account result.** `results` is in the order the accounts were named, and each entry carries `account_id`, `platform`, `success`, `platform_post_url`, `status`, `error`, plus `post_id`/`post` for a post and `thread_id`/`thread` for a thread. Some accounts can succeed while others fail, so report per account rather than declaring the whole send done. A successful entry whose `status` is `processing` has not gone live yet (see below). A validation problem (character limit, media rules, whether the platform takes a first comment, the X link-post rule) refuses the whole call with a message prefixed `Account <id>: ` and publishes nothing; only a platform failure is per account, and the quota for that account is given back.

### chirpie_list_posts

List posts one page at a time. The answer is `{ posts, has_more, next_offset }`: pass `next_offset` back as `offset` while `has_more` is true.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter: draft, scheduled, publishing, processing, published, failed, deleted. `draft` lists posts saved but not sent |
| `from` | string | No | ISO 8601 instant with a timezone. Only posts whose scheduled or publish time is at or after it |
| `to` | string | No | ISO 8601 instant with a timezone. Only posts whose scheduled or publish time is before it |
| `q` | string | No | Words in the text, case-insensitive |
| `platform` | string | No | One network: x, bluesky, linkedin, threads, mastodon, instagram, facebook, telegram |
| `account_id` | string | No | Filter by account |
| `group_id` | string | No | Every post of one multi-account send, by the `group_id` its create call returned |
| `sort` | string | No | `created_at` (default) or `time`: scheduled, else published, else created |
| `order` | string | No | `desc` (default) or `asc` |
| `limit` | number | No | Posts per page, 1 to 100 (default 20) |
| `offset` | number | No | Posts to skip: the `next_offset` from the previous page |
| `include_hidden` | boolean | No | Include posts the user hid with `chirpie_hide_post`. Off by default on every filter |

A post the user deleted stays in the listing with `status: "deleted"` and no cancel reason: Chirpie never removes a post's history. Every post carries `hidden`.

**A post can come back `processing`.** Instagram and Threads process a post's media before it can be published, and so do LinkedIn and Bluesky for a video; a video can take several minutes. When the platform is still working, `chirpie_post` answers with `status: "processing"` instead of `published`. With `account_ids` this is per account: each entry in `results` has its own `status`, and one can be `processing` while another failed, so report each account rather than one status for the whole call. Do not post it again: Chirpie publishes it on its own as soon as the platform is done, usually within a few minutes, or marks it `failed` with the platform's reason (after a refusal, or 30 minutes) and returns its quota (except a LinkedIn post LinkedIn did not confirm, which may be live and stays counted). Tell the user it is on its way, and check it with `chirpie_get_post` a minute or two later. Deleting a `processing` post cancels it and returns its quota, unless Chirpie is publishing it at that moment: then `chirpie_delete_post` answers `409 post_publishing`. Do not report it cancelled; wait, check it with `chirpie_get_post`, and delete again once its status is `published` or `failed` (it can stay `processing` for a few minutes while Chirpie confirms whether the platform took it). A published post is then deleted from the platform, except where Chirpie cannot delete on that platform or account (for example an Instagram account connected through Instagram), which answers `delete_unsupported`: then tell the user to delete it in the platform's app.

A scheduled post becomes `published` (or `failed`, or `deleted` if cancelled) once its time passes, so a post queued yesterday for this morning is no longer under `status: "scheduled"`. That does not mean it was lost: find it by its time with `from`/`to`, which match a post whose scheduled time or publish time falls in the range, whatever its status (drafts excepted: add `status=draft` for those). If a post seems missing, search by its time and text before creating it again: re-sending the same text to the same account within 12 hours is refused with `409 duplicate_post`, which names the post already there.

### chirpie_get_post

Get a single post by ID.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID, from `chirpie_list_posts` or the answer that created the post. Also accepted as `post_id` |

### chirpie_update_post

Edit a post that has not published yet, or finish a draft. Leaving `schedule_at` out keeps the time a queued post already has, so this never publishes one. Only a post that has not published yet can be edited, a scheduled post or a draft: one that is publishing, processing, published, failed or deleted answers `409 post_not_editable`, and so does a draft that has already been promoted. Editing does not count against the monthly quota. Rescheduling one post of a thread moves every part of it, and the response lists them in `rescheduled_post_ids`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID, from `chirpie_list_posts` or the answer that created the post. Also accepted as `post_id` |
| `text` | string | No | Replacement text |
| `media` | object[] | No | Replacement media, each `{ id? , url?, alt? }`. An empty array removes the media |
| `media_urls` | string[] | No | Replacement media URLs. An empty array removes the media |
| `first_comment` | string | No | A new first comment, or an empty string to remove the one the post carries. This is also how a first comment is added to a post that has not published yet. On a thread it belongs to the thread, so it applies whichever part you addressed |
| `configuration` | object | No | New publishing options, or an empty object to put the post back to a plain feed post. A block replaces the post's options rather than merging into them, and the media is checked again against the new placement |
| `schedule_at` | string | No | New ISO 8601 publish time, in the future. Either absolute, carrying a timezone (`...Z` or `+02:00`), or a local time with no offset (`2026-11-01T09:30:00`) read in `timezone` or the timezone saved on the account. A local time with neither is refused. On a draft it promotes: the draft becomes a scheduled post, unless `draft` is true |
| `timezone` | string | No | The IANA zone a `schedule_at` with no offset is read in, such as `America/New_York`. Daylight saving is resolved for the date named, which is what a client computing today's offset gets wrong across a clock change. Ignored when `schedule_at` already carries an offset. Leave it out to use the timezone saved on the account. A fixed offset like `+02:00` is NOT accepted here |
| `draft` | boolean | No | Keep a draft a draft, so `schedule_at` only changes the time it remembers |
| `publish` | boolean | No | Publish a draft now. Only on a draft, and never together with `schedule_at` |
| `allow_duplicate` | boolean | No | Go ahead even when the account already has a post with the same text within 12 hours, when promoting a draft or changing a queued post's text or time |

### The first comment

`first_comment` publishes one comment under the post the moment it goes out, the "link in the first comment" pattern. The comment is posted by the same account, recorded as one of the user's own comments, and appears in the post's comment thread with `own: true`.

Honoured on X, Threads, Instagram and Facebook. Anywhere else the call is refused with `400 first_comment_unsupported`, naming the platform and the four that work: it is never silently dropped.

On a multi-account call the shared `first_comment` reaches every account unless its `account_configurations` entry says otherwise: an entry naming a `first_comment` replaces it for that account, and one setting `"first_comment": ""` publishes that account with none. The empty string is how one call sends a first comment to the accounts that take one while an account whose platform has none still publishes the post. The character limit is the account's own post limit, and a first comment counts as one post against the monthly quota, exactly as a reply does. On X a first comment containing a link carries the same $0.25 charge a link post does, and is refused on the Free plan before the post is published, so nothing goes out.

Every post carries `first_comment` back, either `null` or `{ text, status, comment_id, error }`, where `status` is `pending`, `posted` or `failed`. `pending` means the post has not published yet, or the comment is on its way, and `comment_id` is the comment's id in the post's comment thread.

**A failed first comment never fails its post.** The post publishes, `status` is `failed` and `error` says why. Tell the user, and offer `chirpie_retry_first_comment`.

### Publishing options

`configuration` says how a platform publishes the post, keyed by platform slug. Instagram takes a feed post (the default), a story or a reel; a Facebook Page takes a feed post or a story. Every other platform publishes one way and takes no block.

Instagram's feed posts carry `collaborators` (up to 3 usernames, without the `@`) and `user_tags` (up to 20 `{ username, x, y }`, both coordinates required). A story carries `user_tags` only, with both coordinates or neither. A reel carries `collaborators`, `user_tags` without coordinates, `cover` (an uploaded image id) or `video_cover_timestamp_ms` (never both), `share_to_feed` and `trial_reel: { graduation: "manual" | "performance" }`. A Facebook feed post carries `link`; a Page story carries nothing beyond `placement`.

A story takes exactly one image or video, no caption (send empty text) and no first comment, on both platforms. A reel takes exactly one video. Anything the platform or the placement does not carry is refused with `400 configuration_unsupported` naming the field, and nothing publishes. A story or a reel is a single post, so `chirpie_thread` refuses either placement. Aspect ratio and video duration are the platform's own checks, which come back as `502 upstream_error` at publish time. On Instagram, deleting a published post works on an account connected via Facebook and is refused with `501 delete_unsupported` on one connected through Instagram. Facebook publishes no delete for a Page story, so a published story answers `501 delete_unsupported` and expires on its own after 24 hours.

Full reference: https://chirpie.ai/docs/platforms/instagram?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp and https://chirpie.ai/docs/platforms/facebook?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp

### chirpie_retry_first_comment

Post a first comment that failed, again. It re-sends the text the post already holds. That text cannot be changed once the post is out, because `chirpie_update_post` refuses a published post with `409 post_not_editable`, so set it while the post is still a draft or still queued.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID, from `chirpie_list_posts` or the answer that created the post. Also accepted as `post_id` |
| `idempotency_key` | string | No | Makes retrying this call safe. The same key with the same request replays the first answer for 24 hours instead of sending it again; the same key with a different request is `422 idempotency_key_reused`; a retry arriving while the first is still running is `409 idempotency_in_progress`, which does not wait, so retry once more to collect the replay. Max 255 characters |

The answer is the post, with `first_comment.status` now `posted`. A retry that works costs one post from the monthly quota, exactly as the first attempt would have. Refusals: `404 first_comment_not_found` when the post has none, `409 first_comment_not_retryable` when it is already posted, the post has not published, or another attempt is already in flight, and the platform's own refusal when the retry fails too.

### Drafts

`draft: true` on `chirpie_post` or `chirpie_thread` saves the content and sends nothing: no platform call, no quota, and a draft never publishes on its own. It is held to far less than a post, so the text may be empty, media may be missing where the platform requires it, a draft thread may be a single part, and a `schedule_at` need not be in the future.

Every draft answer carries `warnings`, always present and empty when nothing would go wrong. Each entry is `{ account_id, platform, code, message }`, and a warning for something that would really be refused carries the same `code` and sentence the send would answer with. Three describe a change rather than a refusal: `thread_not_native` (the parts publish as standalone posts), `x_link_post_billed` (the X link surcharge applies), `schedule_at_in_past` (the remembered time has passed). Report them to the user instead of assuming the draft is ready.

Promote with `chirpie_update_post`: `schedule_at` queues it, `publish: true` sends it now, and they are never valid together. Promotion runs every rule a create runs and takes the quota, so anything refused leaves the draft exactly as it was. The answer is a new post carrying `promoted_from_draft_id`, or `promoted_from_draft_ids` for a draft thread, which is promoted whole. The draft's own id is then gone.

### chirpie_delete_post

Take a post down from the platform. The platform is told first, and the post is reported deleted only once the platform confirms it is gone. Deleting any post of a scheduled thread cancels the whole thread.

**The post is never removed from Chirpie**: it keeps its id and its history with `status: "deleted"`, so `chirpie_get_post` still returns it.

A published TikTok post, a published Facebook Page story, or a post on an Instagram account connected through Instagram rather than via Facebook cannot be deleted, so each is refused with `delete_unsupported`. Tell the user to delete it in the platform's own app (a Page story expires on its own after 24 hours), and offer `chirpie_hide_post` to keep it out of their Chirpie listings.

This reaches the platform and cannot be undone, so confirm with the user first.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID, from `chirpie_list_posts` or the answer that created the post. Also accepted as `post_id` |

### chirpie_hide_post

Hide a post from the user's Chirpie listings. **Nothing reaches the platform**: the post stays exactly as it is, keeps its analytics and its comments, and `chirpie_unhide_post` puts it back. Nothing is charged and no quota moves.

Hide is the answer when the user wants a post out of their way; `chirpie_delete_post` is the answer when they want it taken down, and the two are never the same request. A hidden post is left out of `chirpie_list_posts` unless `include_hidden` is true, and is always readable by id with `chirpie_get_post`.

**Hiding a queued post does not stop it publishing.** Hide only decides what Chirpie shows; the scheduler pays no attention to it. Use `chirpie_delete_post` to stop a scheduled post going out.

A thread, or a multi-account send, is hidden as the one thing it was made as, so `hidden_ids` can name more posts than the id you passed.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID. A thread or fan-out moves whole |

### chirpie_unhide_post

Put a hidden post back in the user's Chirpie listings. The exact undo of `chirpie_hide_post`, and as with hide, nothing reaches the platform. Unhiding a post that was never hidden succeeds and changes nothing.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Post UUID. A thread or fan-out moves whole |

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
Setup guide: https://chirpie.ai/docs/x-byo-keys?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp

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
[account settings](https://chirpie.ai/dashboard/accounts?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp) instead. Setting keys
and checking their status stay available over OAuth.

### chirpie_analytics

Get engagement metrics for a published post.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `post_id` | string | Yes | Post UUID. Also accepted as `id`, the name the post tools use |
| `refresh` | boolean | No | Ask the platform for the current numbers instead of reading the stored snapshot. Allowed once per post every 5 minutes; past that it answers `429 analytics_refresh_rate_limited` with a `Retry-After`, and the stored numbers are still one ordinary call away |

Returns: impressions, likes, retweets, replies, quotes, bookmarks, clicks. The numbers come
from a snapshot at most an hour old, so calling this often costs nothing. Reserve `refresh`
for the moment somebody is actually looking at the answer.

### Comments: chirpie_list_comments, chirpie_reply_to_comment, chirpie_hide_comment, chirpie_delete_comment

`chirpie_list_comments` takes `post_id`, `limit`, `cursor`, `since`, `sync`, and two filters:
`include_hidden` adds the comments the user hid (`hidden: true`), and `include_deleted` adds
the ones that are gone (`deleted: true`, with the last text Chirpie stored, so it can still
say what was removed). Both are off by default. The user's own replies carry `own: true` and
are authored by the connected account. Check `capabilities` before offering an action.

`chirpie_delete_comment` deletes on the platform first. On Facebook, Instagram, Threads and
LinkedIn the replies under the comment are deleted with it and read `deleted: true` too; on
X, Bluesky and Mastodon they stay live. Confirm with the user before deleting.

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
- "Save this as a draft, I will decide on the wording later"
- "Show me my drafts, then publish the one about the launch"
- "List my connected accounts"
- "Delete the post with ID xyz"

## Connecting social accounts from chat

The user does not need to leave the agent to connect a platform. Call the matching
`chirpie_connect_*` tool and hand back the `authorization_url` for them to open. It is a
chirpie.ai link: in their browser, Chirpie asks them to sign in if that browser is not
signed in yet, **Continue** takes them to the platform to approve, and they land on a page
saying the account is connected. Then `chirpie_list_accounts` returns its `account_id`.
Hand the link over exactly as it came back: it works for 24 hours, only for this Chirpie
account, and an edited link is refused. If it expires, call the tool again for a new one.
Bluesky and Telegram connect within the call, from the credentials passed to it.

A new Chirpie account has no social account, and nothing publishes until one is
connected. When `chirpie_list_accounts` returns no accounts it carries `next_step`
naming the connect tools, and `chirpie_post` / `chirpie_thread` answer
`no_connected_accounts` with the same pointer. Connecting an account is the first step
for a user who signed in through a connector (Claude, ChatGPT) and has never used the
dashboard.

| Tool | Platform | Arguments |
|------|----------|-----------|
| `chirpie_connect_x` | X/Twitter | none |
| `chirpie_connect_linkedin` | LinkedIn profile | none |
| `chirpie_connect_linkedin_pages` | LinkedIn Pages (coming soon) | none |
| `chirpie_connect_threads` | Threads | none |
| `chirpie_connect_instagram` | Instagram | `via`: `instagram` (the default) signs in with Instagram, `facebook` signs in with Facebook and connects the Instagram accounts linked to the Pages shared, several at once. Only the `facebook` route can delete a published post; everything else is identical. `reconnect`: re-ask about anything turned down last time |
| `chirpie_connect_facebook` | Facebook Pages | none |
| `chirpie_connect_bluesky` | Bluesky | `identifier`, `app_password` (an app password shaped `xxxx-xxxx-xxxx-xxxx`, never the account password; anything else is `bluesky_app_password_required`) |
| `chirpie_connect_mastodon` | Mastodon | `instance_url`: the server, the user's handle (`@you@mastodon.social`) or their profile URL; only the server is kept |
| `chirpie_connect_telegram` | Telegram | `bot_token`, `chat_id` |

## Key management tools

`chirpie_create_key` (returns the key once), `chirpie_list_keys`, `chirpie_revoke_key`.

`chirpie_create_key` takes an optional `scopes` array, which narrows what the new key may do:
`posts:read`, `posts:write`, `accounts:read`, `accounts:write`, `analytics:read`,
`comments:read`, `comments:write`, `media:write`, `keys:write`. Leave it out for a key that
can do exactly what the calling key can do, which is full access for every key that predates
scopes. A key can never grant a scope it does not itself hold, and leaving `scopes` out is
not a way around that: a call outside a key's scopes is refused with `403 insufficient_scope`
naming the one that is missing. There is deliberately no `keys:read`: all three key tools
need `keys:write`, because the key list is the inventory of the account's credentials.

These are API-key auth only, along with `chirpie_remove_x_keys`. On a hosted
server signed in with OAuth they are not offered. Manage API keys in the
[dashboard](https://chirpie.ai/dashboard/keys?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp) instead.

## Authentication Priority

**Hosted server** (either):
1. Sign in through the client's connector flow (recommended)
2. `Authorization: Bearer chirpie_sk_...` header

### Read only or full access

Signing in shows a Chirpie screen asking what the app may do: **Full access** (default)
or **Read only**. A read-only connection holds `posts:read`, `accounts:read`,
`analytics:read` and `comments:read`, and `tools/list` offers only
`chirpie_list_posts`, `chirpie_get_post`, `chirpie_list_accounts`, `chirpie_analytics`,
`chirpie_list_comments` and `chirpie_get_x_keys_status`. Calling anything else answers
`insufficient_scope` naming the missing permission (for example `posts:write`). If the user
asks a read-only connection to publish, tell them to disconnect Chirpie in their AI app,
connect it again and choose Full access. A scoped API key is filtered the same way: only
the tools its scopes cover are offered. Docs: https://chirpie.ai/docs/mcp?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-mcp#read-only-or-full-access

**Local server**:
1. `CHIRPIE_API_KEY` environment variable
2. `~/.chirpie/config.json` (created by `chirpie login`)

Both CLI and the local MCP server share the same config, so `chirpie login` authenticates both.

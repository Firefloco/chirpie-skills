---
name: chirpie-openclaw
description: Post, thread, and schedule to X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram via the Chirpie API.
metadata: { "openclaw": { "requires": { "env": ["CHIRPIE_API_KEY"], "bins": ["curl"] }, "primaryEnv": "CHIRPIE_API_KEY", "envVars": [{ "name": "CHIRPIE_API_KEY", "required": true, "description": "Chirpie API key from https://chirpie.ai/dashboard/keys (starts with chirpie_sk_)." }], "emoji": "🐦", "homepage": "https://chirpie.ai" } }
---

# Chirpie for OpenClaw

Give this agent a social media account. Chirpie is a single API for posting to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram, with threads, scheduling, deletion, and analytics.

**Base URL:** `https://chirpie.ai/api/v1`
**Auth:** `Authorization: Bearer $CHIRPIE_API_KEY`
**Docs:** https://chirpie.ai/docs

## Setup

1. Sign up at https://chirpie.ai/auth/signup
2. Connect social accounts at https://chirpie.ai/dashboard/accounts
3. Create an API key at https://chirpie.ai/dashboard/keys
4. Export it: `export CHIRPIE_API_KEY=chirpie_sk_...`

Install this skill:

```bash
openclaw skills install git:Firefloco/chirpie-skills
openclaw skills install git:Firefloco/chirpie-skills --global   # all agents
openclaw skills install git:Firefloco/chirpie-skills             # re-run to update
```

Prefer tool calls over HTTP? Connect the MCP server instead (same key, same accounts):

```bash
openclaw mcp add chirpie --url https://chirpie.ai/mcp --transport streamable-http
openclaw mcp doctor chirpie --probe
```

## Always start here: resolve the account ID

Posts are addressed by Chirpie's account UUID, never by a handle. List accounts once and reuse the UUIDs.

```bash
curl -s https://chirpie.ai/api/v1/accounts \
  -H "Authorization: Bearer $CHIRPIE_API_KEY"
```

Each account returns `id`, `platform`, `username`, `display_name`, `max_post_length`, and `is_active`. Use `max_post_length`. It is authoritative and already accounts for X Premium.

If `is_active` is `false`, tell the user to re-authorize at https://chirpie.ai/dashboard/accounts. Do not retry.

## Create a post

```bash
curl -s -X POST https://chirpie.ai/api/v1/posts \
  -H "Authorization: Bearer $CHIRPIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "ACCOUNT_UUID",
    "text": "Post text here"
  }'
```

| Field | Required | Notes |
|---|---|---|
| `account_id` | yes | UUID from `GET /accounts` |
| `text` | yes | Must fit the platform limit |
| `media_urls` | no | Array of **public** image/video URLs |
| `schedule_at` | no | Future ISO 8601 timestamp with a timezone (`...Z` or `+02:00`); normalized to UTC |

Returns `201` with the post object. `status` is `published` or `scheduled`.

## Create a thread

2–25 posts. X, Bluesky, Threads, Mastodon, and Telegram thread natively; LinkedIn, Instagram, and Facebook publish each item standalone.

```bash
curl -s -X POST https://chirpie.ai/api/v1/threads \
  -H "Authorization: Bearer $CHIRPIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "ACCOUNT_UUID",
    "posts": [
      { "text": "First post" },
      { "text": "Second post" }
    ]
  }'
```

A thread counts as N posts against the monthly quota.

## Schedule

Add `schedule_at` to either endpoint. Publishes within ~5 minutes of the target time.

```json
{ "account_id": "ACCOUNT_UUID", "text": "Goes out later", "schedule_at": "2026-10-01T09:00:00Z" }
```

Rules: must be in the future, UTC only, and at least **5 minutes apart** from any other scheduled post on the same account.

## List, inspect, delete

```bash
# Queued
curl -s "https://chirpie.ai/api/v1/posts?status=scheduled&limit=50" \
  -H "Authorization: Bearer $CHIRPIE_API_KEY"

# One post
curl -s https://chirpie.ai/api/v1/posts/POST_ID \
  -H "Authorization: Bearer $CHIRPIE_API_KEY"

# Delete (also removes it from the platform, except Instagram and TikTok)
curl -s -X DELETE https://chirpie.ai/api/v1/posts/POST_ID \
  -H "Authorization: Bearer $CHIRPIE_API_KEY"
```

Deleting a scheduled post before its time cancels it.

## Analytics

```bash
curl -s https://chirpie.ai/api/v1/analytics/posts/POST_ID \
  -H "Authorization: Bearer $CHIRPIE_API_KEY"
```

Returns impressions, likes, reposts, replies, quotes, bookmarks, and clicks. Telegram has no metrics API, so Telegram posts return nothing.

## Platform rules to respect

| Platform | Max text | Media |
|---|---|---|
| X | 280 (25,000 Premium) | 4 images or 1 video |
| Bluesky | 300 | 4 images, no video |
| LinkedIn | 3,000 | 4 images, no video |
| Threads | 500 | 1 image, no video |
| Mastodon | 500 | 4 images or 1 video |
| Instagram | 2,200 | **image required**; 2–10 = carousel |
| Facebook | 63,206 | 4 images; Pages only |
| Telegram | 4,096 | 10 images or 1 video |

Media URLs must be publicly reachable, because Chirpie downloads them server-side. `localhost` and short-lived signed URLs will fail.

## Response format

```json
{ "data": { } }
{ "error": { "code": "error_code", "message": "Human-readable description" } }
```

| Status | Code | Meaning | What to do |
|---|---|---|---|
| 400 | `bad_request` | Text too long, past `schedule_at`, spacing violation | Fix and retry once |
| 401 | `unauthorized` | Bad or revoked key | Stop; tell the user |
| 404 | `not_found` | Wrong or inactive `account_id` | Re-list accounts |
| 429 | `rate_limited` | Monthly quota or 60 req/min burst | Stop; tell the user |
| 502 | `upstream_error` | Platform API failed | Retry once, then report |

## Behaviour rules for this agent

1. **Never invent an `account_id`.** Always list accounts first.
2. **Check the length** against `max_post_length` before posting. Rewrite rather than truncate mid-word.
3. **Never post text-only to Instagram.** Ask for an image.
4. **Prefer scheduling** for anything the user has not explicitly approved, since it stays cancellable.
5. **Confirm before deleting.** Deletion removes the post from the live platform too.
6. **Do not echo the API key** into logs or output.
7. **Report failures plainly** with the `code` and `message` from the response. Do not silently retry `401` or `429`.

## Example requests

- "Post to Bluesky: the nightly job finished clean."
- "Turn today's commits into a 4-post X thread."
- "Schedule a LinkedIn post for 9am UTC Monday."
- "What's queued for next week?"
- "Cancel Thursday's post."
- "How did yesterday's post do?"

---
name: chirpie
description: Chirpie social media API router. Use when user asks about posting to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, or Telegram, social media automation, scheduling posts, first comments, saving drafts, connecting social accounts, or using the Chirpie API/SDK/CLI/MCP/n8n node. Automatically routes to the specific skill based on their task.
---

# Chirpie Skills Router

Chirpie is a social media API for AI agents and developers. Post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram via API, CLI, MCP, or SDK, with scheduling, analytics, and multi-account support.

**Base URL:** `https://chirpie.ai/api/v1`
**Auth:** `Authorization: Bearer chirpie_sk_YOUR_KEY`
**Docs:** https://chirpie.ai/docs

## By Task

**Setting up Chirpie in a project** → Use `chirpie-setup`
- Install the SDK
- Configure API keys
- Connect accounts
- First post quickstart

**Creating posts or threads** → Use `chirpie-posting`
- Single posts (immediate)
- Multi-post threads (2-25 posts)
- Post listing, retrieval, deletion
- Analytics and metrics

**Reading or answering comments on a post** → Use `chirpie-posting`
- List the comments a published post received
- Reply to a comment (counts as one post against the monthly quota)
- Hide or delete a comment, where the platform allows it

**Putting the link in the first comment** → Use `chirpie-posting`
- `first_comment` on a post or a thread, published under the post the moment it goes out
- X, Threads, Instagram and Facebook only, and it counts as one post against the monthly quota
- Sending a first comment that failed again with `POST /api/v1/posts/:id/first-comment`

**Publishing an Instagram story or reel, or a Facebook Page story** → Use `chirpie-posting`
- `configuration` on a post, keyed by platform: Instagram takes feed, story or reel, a Facebook Page takes feed or story
- Collaborators, user tags, reel covers, trial reels, a Facebook link preview
- Connecting a new Instagram account or Facebook Page is coming soon; accounts already connected publish as described

**Connecting an Instagram account** → Use `chirpie-setup`
- Two routes: sign in with Instagram (the default), or sign in with Facebook and connect the Instagram accounts linked to the Pages shared
- Identical for posting, threads, scheduling and analytics; only the Facebook route can delete a published post

**Posting the same thing to several accounts at once** → Use `chirpie-posting`
- `account_ids` (1-25) instead of `account_id` on posts and threads
- Per-account text and media with `account_configurations`
- Reading the `group_id` and per-account `results` back

**Scheduling content for later** → Use `chirpie-scheduling`
- Schedule posts and threads
- Understand timing, retries, and limits
- Cancel scheduled posts

**Saving a draft, or finishing one** → Use `chirpie-scheduling`
- `draft: true` on a post or thread saves it without sending it
- The `warnings` a draft comes back with
- Promoting it: `schedule_at` queues it, `publish: true` sends it now

**Using the TypeScript SDK** → Use `chirpie-sdk`
- Install and configure `@chirpie/sdk`
- All client methods and types
- Error handling patterns

**Using the CLI** → Use `chirpie-cli`
- Install and authenticate
- Post from the terminal
- Manage accounts and keys

**Setting up the MCP server** → Use `chirpie-mcp`
- Configure for Claude Code, Cursor, or Claude Desktop
- Available tools and parameters
- Authentication flow

**Using Chirpie in n8n** → See https://chirpie.ai/docs/n8n
- Install the `@chirpie/n8n-nodes-chirpie` community node (Settings → Community Nodes → `@chirpie/n8n-nodes-chirpie`)
- Add a Chirpie API credential, then use the Chirpie node for posts, threads, accounts, and analytics
- The node is usable as a tool by n8n AI Agents

**Running inside OpenClaw** → Use `chirpie-openclaw`
- Install via `openclaw skills install`
- Connect the MCP server with `openclaw mcp add`
- Behaviour rules for autonomous posting agents

## Quick Reference

| Action | Endpoint | Method |
|--------|----------|--------|
| Create post | `/api/v1/posts` | POST |
| List posts (also by `group_id`) | `/api/v1/posts` | GET |
| Get post | `/api/v1/posts/:id` | GET |
| Update post | `/api/v1/posts/:id` | PATCH |
| Delete post | `/api/v1/posts/:id` | DELETE |
| Create thread | `/api/v1/threads` | POST |
| Retry first comment | `/api/v1/posts/:id/first-comment` | POST |
| Upload media | `/api/v1/media` | POST |
| List accounts | `/api/v1/accounts` | GET |
| Connect account | `/api/v1/accounts` | POST |
| Create API key | `/api/v1/keys` | POST |
| List keys | `/api/v1/keys` | GET |
| Revoke key | `/api/v1/keys?id=ID` | DELETE |
| Post analytics | `/api/v1/analytics/posts/:id` | GET |
| List comments | `/api/v1/posts/:id/comments` | GET |
| Reply to comment | `/api/v1/posts/:id/comments/:comment_id/reply` | POST |
| Hide comment | `/api/v1/posts/:id/comments/:comment_id/hide` | POST |
| Delete comment | `/api/v1/posts/:id/comments/:comment_id` | DELETE |

## Response Format

All endpoints return:

```json
// Success
{ "data": { ... } }

// Error
{ "error": { "code": "error_code", "message": "Human-readable description" } }
```

## Rate Limits

- **Burst:** per API key, per minute, on a sliding window, with the ceiling set by the plan: Free 120/min, Agent and Starter 600/min, Pro and Scale 1,200/min. Read `X-RateLimit-Limit` rather than assuming a number, and pace on `X-RateLimit-Remaining`. A `429 rate_limited` carries `Retry-After`.
- **Monthly quotas:** Free 50, Agent 300, Starter 1K, Pro 5K, Scale 25K+ / custom posts per month
- **Overage:** $0.03/post (Agent, Starter), $0.025/post (Pro). Free plan has a hard limit. Scale and Enterprise use custom quotas, not per-post overage.
- **Comment syncs:** refreshing a post's comments from the platform is metered per month: Free 200, Agent 1,000, Starter 5,000, Pro 25,000, Scale and Enterprise custom. Listing comments Chirpie already stored is unlimited, and a reply counts as one post against the monthly post quota. X comment reads are metered per reply returned as well, and are not included on Free.
- **X link posts:** X charges API operators $0.20 per post containing a URL. Paid plans are billed $0.25 per X link post on top of the monthly allowance; on Free, X posts containing links are rejected with `402 x_link_posts_require_paid_plan`. X accounts connected with your own X API credentials are exempt, since you pay X directly. Other platforms are unaffected.

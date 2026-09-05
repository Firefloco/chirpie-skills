---
name: chirpie
description: Chirpie social media API router. Use when user asks about posting to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, Telegram, Pinterest, TikTok, YouTube, or Google Business Profile, social media automation, scheduling posts, connecting social accounts, or using the Chirpie API/SDK/CLI/MCP/n8n node. Automatically routes to the specific skill based on their task.
---

# Chirpie Skills Router

Chirpie is a social media API for AI agents and developers. Post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram via API, CLI, MCP, or SDK — with scheduling, analytics, and multi-account support.

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

**Scheduling content for later** → Use `chirpie-scheduling`
- Schedule posts and threads
- Understand timing, retries, and limits
- Cancel scheduled posts

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
| List posts | `/api/v1/posts` | GET |
| Get post | `/api/v1/posts/:id` | GET |
| Delete post | `/api/v1/posts/:id` | DELETE |
| Create thread | `/api/v1/threads` | POST |
| List accounts | `/api/v1/accounts` | GET |
| Connect account | `/api/v1/accounts` | POST |
| Create API key | `/api/v1/keys` | POST |
| List keys | `/api/v1/keys` | GET |
| Revoke key | `/api/v1/keys?id=ID` | DELETE |
| Post analytics | `/api/v1/analytics/posts/:id` | GET |

## Response Format

All endpoints return:

```json
// Success
{ "data": { ... } }

// Error
{ "error": { "code": "error_code", "message": "Human-readable description" } }
```

## Rate Limits

- **Burst:** 60 requests/minute per API key
- **Monthly quotas:** Free 50, Agent 300, Starter 1K, Pro 5K, Scale 25K+ / custom posts per month
- **Overage:** $0.03/post (Agent, Starter), $0.025/post (Pro). Free plan has a hard limit. Scale and Enterprise use custom quotas, not per-post overage.
- **X link posts:** X charges API operators $0.20 per post containing a URL. Paid plans are billed $0.25 per X link post on top of the monthly allowance; on Free, X posts containing links are rejected with `402 x_link_posts_require_paid_plan`. X accounts connected with your own X API credentials are exempt — you pay X directly. Other platforms are unaffected.

---
name: chirpie
description: Chirpie social media API router. Use when user asks about posting to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, or Facebook, social media automation, scheduling posts, connecting X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, or Facebook accounts, or using the Chirpie API/SDK/CLI/MCP. Automatically routes to the specific skill based on their task.
---

# Chirpie Skills Router

Chirpie is a social media API for AI agents and developers. Post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, and Facebook via API, CLI, MCP, or SDK — with scheduling, analytics, and multi-account support.

**Base URL:** `https://chirpie.ai/api/v1`
**Auth:** `Authorization: Bearer chirpie_sk_YOUR_KEY`
**Docs:** https://chirpie.ai/docs

## By Task

**Setting up Chirpie in a project** → Use `chirpie-setup`
- Install the SDK
- Configure API keys
- Connect X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, and Facebook accounts
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
- **Monthly quotas:** Free 50, Starter 1K, Pro 5K, Scale 25K posts/month
- **Overage:** $0.01/post (Starter), $0.008/post (Pro). Free plan has hard limit. Scale is custom pricing.

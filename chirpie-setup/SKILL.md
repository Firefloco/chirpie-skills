---
name: chirpie-setup
description: Set up Chirpie in your project — install SDK, configure API keys, connect X, Bluesky, LinkedIn, Threads, and Mastodon accounts, send your first post.
---

# Chirpie Setup

## Prerequisites

- A Chirpie account at https://chirpie.ai/auth/signup
- An API key (created during onboarding or at https://chirpie.ai/dashboard/keys)
- At least one connected X, Bluesky, LinkedIn, Threads, or Mastodon account (via https://chirpie.ai/dashboard/accounts)

## Step 1: Install the SDK

```bash
npm install @chirpie/sdk
```

## Step 2: Configure Your API Key

**Option A: Environment variable (recommended)**

```bash
# .env or .env.local
CHIRPIE_API_KEY=chirpie_sk_YOUR_KEY
```

**Option B: Direct initialization**

```typescript
import { ChirpieClient } from "@chirpie/sdk";

const chirpie = new ChirpieClient({
  apiKey: process.env.CHIRPIE_API_KEY!,
});
```

> **Security:** Never hardcode API keys in source code. Always use environment variables.

## Step 3: Connect Accounts

**X/Twitter** — Connect via OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectAccount();
// Open URL in browser to authorize
```

**Bluesky** — Connect with an app password (generate at https://bsky.app/settings/app-passwords):
```typescript
await chirpie.connectBlueskyAccount({
  platform: "bluesky",
  identifier: "yourhandle.bsky.social",
  app_password: "xxxx-xxxx-xxxx-xxxx",
});
```

Or via CLI:
```bash
chirpie accounts connect-bluesky --handle yourhandle.bsky.social --app-password xxxx-xxxx-xxxx-xxxx
# --handle is required, --app-password is optional (prompts securely if omitted)
```

**LinkedIn** — Connect via OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectLinkedInAccount();
// Open URL in browser to authorize via LinkedIn OAuth
```

Or via CLI:
```bash
chirpie accounts connect-linkedin
# Opens browser for LinkedIn OAuth authorization
```

**Threads** — Connect via Meta OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectThreadsAccount();
// Open URL in browser to authorize via Meta OAuth
```

Or via CLI:
```bash
chirpie accounts connect-threads
# Opens browser for Meta OAuth authorization
```

**Mastodon** — Connect via OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectMastodonAccount({
  platform: "mastodon",
  instance_url: "https://mastodon.social",
});
// Open URL in browser to authorize via Mastodon OAuth
```

Or via CLI:
```bash
chirpie accounts connect-mastodon --instance https://mastodon.social
# Opens browser for Mastodon OAuth authorization
```

## Step 4: Find Your Account ID

```typescript
const accounts = await chirpie.listAccounts();
console.log(accounts);
// Each account has: id, platform ("x", "bluesky", "linkedin", "threads", or "mastodon"), username, display_name, avatar_url
// Use the `id` field as your account_id for posting
```

## Step 5: Send Your First Post

```typescript
const post = await chirpie.createPost({
  account_id: "YOUR_ACCOUNT_ID",
  text: "Hello from Chirpie! 🐦",
});

console.log(`Posted! Status: ${post.status}`);
```

## Alternative: CLI Setup

```bash
# Install globally
npm install -g chirpie

# Login via browser (creates API key automatically)
chirpie login

# Post (auto-selects account if you only have one)
chirpie post "Hello from the CLI!"
```

## Alternative: MCP Setup

See `chirpie-mcp` skill for Claude/Cursor/AI agent configuration.

## Common Pitfalls

1. **API key format:** Keys must start with `chirpie_sk_`. If you get 401 errors, check the prefix.
2. **Account not active:** If OAuth tokens expired (X, LinkedIn, Threads), token was revoked (Mastodon), or app password is revoked (Bluesky), `is_active` will be `false`. Re-authorize from the dashboard.
3. **Environment variable loading:** In Next.js, server-side env vars don't need `NEXT_PUBLIC_` prefix. Don't expose your API key to the browser.

## Plan Limits

| Plan | Posts/mo | Scheduled/mo | Accounts | Price |
|------|----------|-------------|----------|-------|
| Free | 50 | 25 | 1 | $0 |
| Starter | 1,000 | 500 | 3 | $19/mo |
| Pro | 5,000 | 2,500 | 10 | $49/mo |
| Scale | 25,000+ | 12,500+ | 25+ | Custom |

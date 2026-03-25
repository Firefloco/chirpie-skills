---
name: chirpie-setup
description: Set up Chirpie in your project — install SDK, configure API keys, connect X accounts, send your first post.
---

# Chirpie Setup

## Prerequisites

- A Chirpie account at https://chirpie.ai/auth/signup
- An API key (created during onboarding or at https://chirpie.ai/dashboard/keys)
- At least one connected X account (via https://chirpie.ai/dashboard/accounts)

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

## Step 3: Find Your Account ID

```typescript
const accounts = await chirpie.listAccounts();
console.log(accounts);
// Use the `id` field as your account_id for posting
```

## Step 4: Send Your First Post

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
2. **Account not active:** If OAuth tokens expired, `is_active` will be `false`. Re-authorize from the dashboard.
3. **Environment variable loading:** In Next.js, server-side env vars don't need `NEXT_PUBLIC_` prefix. Don't expose your API key to the browser.

## Plan Limits

| Plan | Posts/mo | Scheduled/mo | Accounts | Price |
|------|----------|-------------|----------|-------|
| Free | 50 | 25 | 1 | $0 |
| Starter | 1,000 | 500 | 3 | $19/mo |
| Pro | 5,000 | 2,500 | 10 | $49/mo |
| Scale | 25,000+ | 12,500+ | 25+ | Custom |

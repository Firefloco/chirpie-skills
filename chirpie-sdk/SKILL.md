---
name: chirpie-sdk
description: Use the @chirpie/sdk TypeScript client in your application. Covers installation, configuration, all methods, types, and error handling.
---

# Chirpie TypeScript SDK

## Installation

```bash
npm install @chirpie/sdk
```

## Configuration

```typescript
import { ChirpieClient } from "@chirpie/sdk";

// Option 1: Direct configuration
const chirpie = new ChirpieClient({
  apiKey: process.env.CHIRPIE_API_KEY!,
  baseUrl: "https://chirpie.ai",  // Optional, this is the default
});

// Option 2: Auto-load from env or ~/.chirpie/config.json
import { requireConfig } from "@chirpie/sdk";
const config = requireConfig();
const chirpie = new ChirpieClient({
  apiKey: config.api_key,
  baseUrl: config.base_url,
});
```

**Environment variables:**
- `CHIRPIE_API_KEY` — API key (takes precedence over config file)
- `CHIRPIE_BASE_URL` — Override base URL (optional)

**Config file:** `~/.chirpie/config.json` (created by `chirpie login`)

## All Methods

### Posts

```typescript
// Create a post (immediate or scheduled)
const post = await chirpie.createPost({
  account_id: "uuid",        // Required
  text: "Hello!",            // Required. X: 1-280 chars (25,000 for X Premium). Bluesky: 1-300 chars. LinkedIn: 1-3,000 chars. Threads: 1-500 chars. Mastodon: 1-500 chars.
  media_urls: ["url"],       // Optional, max 4
  schedule_at: "ISO8601",    // Optional, must be future
});

// List posts with filters
const posts = await chirpie.listPosts({
  status: "published",       // Optional
  account_id: "uuid",        // Optional
  limit: 20,                 // Optional, max 100
  offset: 0,                 // Optional
});

// Get a single post
const post = await chirpie.getPost("post-uuid");

// Delete a post (also deletes from X/Bluesky/LinkedIn/Threads/Mastodon if published)
const result = await chirpie.deletePost("post-uuid");
```

### Threads

```typescript
const thread = await chirpie.createThread({
  account_id: "uuid",
  posts: [
    { text: "First", media_urls: [] },
    { text: "Second" },
  ],
  schedule_at: "ISO8601",    // Optional
});
```

### Accounts

```typescript
// List all connected accounts (X, Bluesky, LinkedIn, Threads, and Mastodon)
const accounts = await chirpie.listAccounts();
// Each account: { id, platform, username, display_name, avatar_url, is_active }

// Connect X account (OAuth flow)
const { authorization_url } = await chirpie.connectAccount();

// Connect Bluesky account (app password)
await chirpie.connectBlueskyAccount({
  platform: "bluesky",
  identifier: "yourhandle.bsky.social",
  app_password: "xxxx-xxxx-xxxx-xxxx",
});

// Connect LinkedIn account (OAuth flow)
const { authorization_url } = await chirpie.connectLinkedInAccount();

// Connect Threads account (Meta OAuth flow)
const { authorization_url } = await chirpie.connectThreadsAccount();

// Connect Mastodon account (OAuth flow)
const { authorization_url } = await chirpie.connectMastodonAccount({
  platform: "mastodon",
  instance_url: "https://mastodon.social",
});
```

### API Keys

```typescript
const { key, prefix, name } = await chirpie.createKey("My Bot");
const keys = await chirpie.listKeys();
await chirpie.revokeKey("key-uuid");
```

### Analytics

```typescript
const metrics = await chirpie.getPostAnalytics("post-uuid");
// { impressions, likes, retweets, replies, quotes, bookmarks, clicks, fetched_at }
```

## Types

```typescript
import type {
  ApiPost,                    // Post object (includes `platform` field)
  ApiThread,                  // Thread object returned from API
  ApiAccount,                 // Connected account (platform: "x"|"bluesky"|"linkedin"|"threads"|"mastodon", username, display_name, avatar_url)
  ApiAnalytics,               // Post metrics
  ApiKeyInfo,                 // API key metadata (prefix only)
  CreatePostInput,            // Input for createPost()
  CreateThreadInput,          // Input for createThread()
  ConnectBlueskyInput,        // Input for connectBlueskyAccount()
  ConnectLinkedInInput,       // Input for connectLinkedInAccount()
  ConnectThreadsInput,        // Input for connectThreadsAccount()
  ConnectMastodonInput,       // Input for connectMastodonAccount()
  ListPostsOptions,           // Options for listPosts()
} from "@chirpie/sdk";
```

## Error Handling

```typescript
import { ChirpieApiError, ChirpieError } from "@chirpie/sdk";

try {
  await chirpie.createPost({ ... });
} catch (err) {
  if (err instanceof ChirpieApiError) {
    console.error(err.code);     // "rate_limited", "not_found", etc.
    console.error(err.message);  // Human-readable description
    console.error(err.status);   // HTTP status code (400, 401, 404, 429, 502)
  } else if (err instanceof ChirpieError) {
    console.error(err.message);  // Network or config error
  }
}
```

## Config Utilities

```typescript
import { getConfig, saveConfig, deleteConfig, requireConfig } from "@chirpie/sdk";

// Read config (returns null if not found)
const config = getConfig();

// Save config (creates ~/.chirpie/config.json with 0600 permissions)
saveConfig("chirpie_sk_...", "https://chirpie.ai");

// Delete config
deleteConfig();

// Read config or throw (for scripts that require auth)
const config = requireConfig();
```

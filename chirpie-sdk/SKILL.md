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
- `CHIRPIE_API_KEY`: API key (takes precedence over config file)
- `CHIRPIE_BASE_URL`: Override base URL (optional)

**Config file:** `~/.chirpie/config.json` (created by `chirpie login`)

## All Methods

### Posts

```typescript
// Upload a file and attach it. The id is valid for 7 days, and a post keeps
// its own copy of the bytes, so an expired id never breaks a queued post.
const media = await chirpie.uploadMedia({
  data: await readFile("./screenshot.png"),
  filename: "screenshot.png",
});
// Or: await chirpie.uploadMedia({ url: "https://example.com/a.png" })

// Create a post (immediate or scheduled)
const post = await chirpie.createPost({
  account_id: "uuid",        // Required
  text: "Hello!",            // Required. Char limits: X 280 (25,000 Premium), Bluesky 300, LinkedIn 3,000, Threads 500, Mastodon 500, Instagram 2,200, Facebook 63,206, Telegram 4,096
  media: [{ url: "https://example.com/a.png", alt: "A bird" }],  // Optional. An entry takes `id` (from uploadMedia) or `url`, plus optional alt text. Max per post: X 4, Bluesky 4, LinkedIn 4, Threads 1, Mastodon 4, Instagram 10, Facebook 10, Telegram 10. Instagram REQUIRES media.
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

// Edit a post that has not published yet. Leaving `schedule_at` out keeps the
// time it already has, so this never publishes anything.
await chirpie.updatePost("post-uuid", { text: "Now with the typo fixed" });
await chirpie.updatePost("post-uuid", { schedule_at: "2027-04-02T09:00:00Z" });

// Delete a post. Removes it from the platform first, and only then from Chirpie.
// If the platform refuses, nothing changes and the call throws: just retry.
const result = await chirpie.deletePost("post-uuid");
```

### Threads

```typescript
const thread = await chirpie.createThread({
  account_id: "uuid",
  posts: [
    { text: "First", media: [{ id: "media-uuid", alt: "A bird" }] },
    { text: "Second" },
  ],
  schedule_at: "ISO8601",    // Optional
});
```

### Accounts

```typescript
// List all connected accounts, active and inactive
const accounts = await chirpie.listAccounts();
// Each account: { id, platform, username, display_name, avatar_url, is_active }
// Inactive accounts may carry inactive_reason: "plan_limit", meaning they are
// connected but were not switched on because the plan's account limit was full.
// One Facebook authorization can grant several Pages; all of them are stored.
// LinkedIn is the same: your profile plus every Page you administer, each with
// its own id. account_type is "member" for a profile, "organization" for a Page.

// Choose which accounts publish. Deactivating CANCELS the account's scheduled
// posts (a switched-off account cannot publish) and returns their quota;
// `scheduled_posts` on the account says how many would go, `canceled_posts` on
// the result says how many did. Activating again does not restore them.
await chirpie.deactivateAccount(id);  // stays connected, frees a plan slot
await chirpie.activateAccount(id);    // fails if no slot is free

// Disconnect an account for good. It stops publishing straight away, stops
// counting against the plan's account limit, and the stored credential is
// removed, so connecting it again means authorizing it on the platform again.
// It CANCELS the account's scheduled posts too (`canceled_posts` says how many,
// and they are not restored); posts it already published are kept.
// Confirm with the user first: this cannot be undone.
const gone = await chirpie.disconnectAccount(id);  // ApiAccountDisconnect
console.log(gone.disconnected, gone.canceled_posts);

// Connect X account (OAuth flow)
const { authorization_url } = await chirpie.connectXAccount();

// Optional: use your own X developer app so posts bill your X API credits
// (and X link posts are not surcharged). Setup: https://chirpie.ai/docs/x-byo-keys
await chirpie.setXKeys({
  client_id: process.env.X_CLIENT_ID!,
  client_secret: process.env.X_CLIENT_SECRET!,
});
await chirpie.getXKeysStatus();  // { configured, client_id_last4, redirect_uri, ... }
await chirpie.removeXKeys();
// Reconnect each X account afterwards to move it onto your app.

// Connect Bluesky account (app password). The identifier takes the first part
// of the handle alone, a full handle, a custom-domain handle, or the account email.
await chirpie.connectBlueskyAccount({
  platform: "bluesky",
  identifier: "yourhandle.bsky.social",
  app_password: "xxxx-xxxx-xxxx-xxxx",
});

// Connect LinkedIn account (OAuth flow)
const { authorization_url } = await chirpie.connectLinkedInAccount();
// LinkedIn Pages you administer are a separate connection (coming soon):
const { authorization_url: pagesUrl } = await chirpie.connectLinkedInPagesAccount();

// Connect Threads account (Meta OAuth flow, coming soon)
const { authorization_url } = await chirpie.connectThreadsAccount();

// Connect Mastodon account (OAuth flow)
const { authorization_url } = await chirpie.connectMastodonAccount({
  platform: "mastodon",
  instance_url: "https://mastodon.social",
});

// Connect Instagram account (Instagram Login OAuth flow, coming soon)
const { authorization_url } = await chirpie.connectInstagramAccount();

// Connect Facebook Page (Facebook Login OAuth flow, coming soon)
const { authorization_url } = await chirpie.connectFacebookAccount();

// Connect Telegram bot (bot token auth)
await chirpie.connectTelegramAccount({
  platform: "telegram",
  bot_token: "TOKEN",
  chat_id: "CHAT_ID",
});

// Threads, Instagram, Facebook, Pinterest, TikTok, YouTube, and Google Business
// Profile are coming soon. Their
// connect methods exist on the client but reject with `platform_coming_soon`.
```

### API Keys

```typescript
const { key, prefix, name, expires_at } = await chirpie.createKey("My Bot");
// `key` is shown once. Keys expire after 90 days; max 25 active keys.
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
  ApiAccount,                 // Connected account (platform, username, display_name, avatar_url)
  ApiAnalytics,               // Post metrics
  ApiKeyInfo,                 // API key metadata (prefix only)
  CreatePostInput,            // Input for createPost()
  CreateThreadInput,          // Input for createThread()
  ConnectBlueskyInput,        // Input for connectBlueskyAccount()
  ConnectLinkedInInput,       // Input for connectLinkedInAccount()
  LinkedInConnectTarget,      // "profile" | "pages"
  ConnectThreadsInput,        // Input for connectThreadsAccount()
  ConnectMastodonInput,       // Input for connectMastodonAccount()
  ConnectInstagramInput,      // Input for connectInstagramAccount()
  ConnectFacebookInput,       // Input for connectFacebookAccount()
  ConnectTelegramInput,       // Input for connectTelegramAccount()
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
    console.error(err.code);     // "usage_limit_exceeded", "rate_limited", "not_found", etc.
    console.error(err.message);  // Human-readable description
    console.error(err.status);   // HTTP status code (400, 401, 402, 404, 429, 502, 503)
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

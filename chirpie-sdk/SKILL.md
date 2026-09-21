---
name: chirpie-sdk
description: Use the @chirpie/sdk TypeScript client in your application. Covers installation, configuration, all methods including posting to several accounts at once, the first comment, and saving drafts, plus types and error handling.
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
  schedule_at: "ISO8601",    // Optional, must be future. Either an absolute instant carrying a timezone, or a local time with no offset ("2026-11-01T09:30:00") read in `timezone` or the timezone saved on the account
  timezone: "America/New_York",  // Optional IANA name. The zone a `schedule_at` with no offset is read in. Daylight saving is resolved for the date named. A fixed offset like "+02:00" is NOT accepted here: put it on schedule_at instead. Also on CreateThreadInput and UpdatePostInput
  first_comment: "Full write-up: https://example.com",  // Optional. Published under the post the moment it goes out. X, Threads, Instagram and Facebook only; anywhere else the call throws 400 first_comment_unsupported. Counts as one post against the quota
  configuration: { instagram: { placement: "reel", share_to_feed: true } },  // Optional. Per-platform publishing options. Instagram: feed (default), story or reel. Facebook Page: feed or story. An option the platform or the placement does not carry throws 400 configuration_unsupported, never dropped. A story carries no caption and no first comment; a story or a reel is a single post, so createThread refuses either. updatePost takes it too, and {} puts a post back to a plain feed post
});

// Retries. createPost() and createThread() generate an Idempotency-Key per
// call. Pass your own so the key survives a process restart, or null for none.
await chirpie.createPost(
  { account_id: "uuid", text: "Hello!" },
  { idempotencyKey: "campaign-2026-11-01" }
);
// uploadMedia(input, options), retryFirstComment(id, options) and
// replyToComment(postId, commentId, text, options) take the same options
// object but generate nothing.

// The post carries the first comment back:
// post.first_comment = { text, status: "pending" | "posted" | "failed", comment_id, error }
// A failed first comment never fails its post. Send it again with the text the
// post already holds. That text cannot be changed once the post is out, so
// set it while the post is still a draft or still queued, with updatePost.
if (post.first_comment?.status === "failed") {
  await chirpie.retryFirstComment(post.id);
}

// Post to several accounts in one call. Naming `account_ids` (1-25) instead of
// `account_id` returns `{ group_id, results }` instead of a single post, even
// for one account. `results` is in the order the accounts were named, and each
// entry carries `success`, `post_id`, `platform_post_url`, `status`, `post`
// and `error`. Status is 201 when every account succeeded, 207 when one did
// not, so branch on `success`, never on the status code.
const { group_id, results } = await chirpie.createPost({
  account_ids: ["uuid-a", "uuid-b"],
  text: "Shared text",                 // Required: what an account with no override publishes
  account_configurations: {            // Optional, keyed by account id, every key must be in account_ids
    "uuid-b": { text: "Text for just this account" },
  },
});

// List posts with filters
const posts = await chirpie.listPosts({
  status: "published",       // Optional
  account_id: "uuid",        // Optional
  group_id: "uuid",          // Optional: every post of one multi-account send
  limit: 20,                 // Optional, max 100
  offset: 0,                 // Optional
});

// Get a single post
const post = await chirpie.getPost("post-uuid");

// Edit a post that has not published yet. Leaving `schedule_at` out keeps the
// time it already has, so this never publishes a queued post.
await chirpie.updatePost("post-uuid", { text: "Now with the typo fixed" });
await chirpie.updatePost("post-uuid", { schedule_at: "2027-04-02T09:00:00Z" });
// Replace the first comment, or pass "" to remove it. This is also how a first
// comment is added to a post that has not published yet.
await chirpie.updatePost("post-uuid", { first_comment: "Full write-up: https://example.com" });

// Delete a post. Takes it down from the platform first, and reports it deleted
// only once the platform confirms it is gone. If the platform refuses, nothing
// changes and the call throws: just retry. Chirpie keeps the post, marked
// deleted, so it stays in the user's history. Instagram and TikTok publish
// no delete API and throw `501 delete_unsupported`.
const result = await chirpie.deletePost("post-uuid");

// Hide a post from the user's Chirpie listings. Nothing reaches the platform:
// the post stays exactly as it is, and unhidePost puts it back. A thread or a
// multi-account send moves whole, and `hidden_ids` names every post that moved.
await chirpie.hidePost("post-uuid");
await chirpie.unhidePost("post-uuid");

// Hidden posts are left out of every listing unless you ask for them.
await chirpie.listPosts({ include_hidden: true });
```

### Threads

A thread is atomic. If any part fails, every part that had already published is deleted from the platform and the whole thread's quota is refunded. `ChirpieApiError.threadRollback()` says what happened; `upstream_error` means nothing is left on the platform and a retry is safe, while `thread_rollback_incomplete` means the posts in `still_live` are really still up and a retry would publish them twice.

```typescript
import { ChirpieApiError } from "@chirpie/sdk";

try {
  await chirpie.createThread({ account_id: "uuid", posts });
} catch (err) {
  const rollback = err instanceof ChirpieApiError ? err.threadRollback() : null;
  for (const part of rollback?.still_live ?? []) {
    console.error("still live:", part.platform_post_url ?? part.platform_post_id);
  }
}
```

```typescript
const thread = await chirpie.createThread({
  account_id: "uuid",
  posts: [
    { text: "First", media: [{ id: "media-uuid", alt: "A bird" }] },
    { text: "Second" },
  ],
  schedule_at: "ISO8601",    // Optional
  first_comment: "Full write-up: https://example.com",  // Optional. One comment for the whole thread, published under the LAST part and reported on that part
});

// The same thread to several accounts. An override's `posts` replaces the whole
// array for that account (still 2-25 parts). Result entries carry `thread_id`
// and `thread` where a post fan-out carries `post_id` and `post`.
const { group_id, results } = await chirpie.createThread({
  account_ids: ["uuid-a", "uuid-b"],
  posts: [{ text: "one" }, { text: "two" }],
  account_configurations: {
    "uuid-b": { posts: [{ text: "a" }, { text: "b" }, { text: "c" }] },
  },
});
```

A multi-account send reserves quota once for the whole group (one unit per
account for a post, one per part per account for a thread). If it does not fit
the plan the call throws `429 usage_limit_exceeded` and nothing publishes.
Validation (character limits, media rules, whether the platform takes a first
comment, the X link-post rule) is run for every account up front, so a failure there refuses the whole request with a
message prefixed `Account <id>: `. Only the platform call itself fails per
account, and those land in `results[].error`.

### Drafts

```typescript
// Save a post without sending it. Nothing reaches the platform, nothing counts
// against either quota, and the text may even be empty.
const draft = await chirpie.createPost({
  account_id: "uuid",
  text: "Half an idea. Finish it later.",
  schedule_at: "2027-04-01T14:00:00Z",  // Optional, and only a time to remember
  draft: true,
});
// draft.status === "draft". `warnings` is always present, empty when nothing
// would go wrong: { account_id, platform, code, message } per problem.
for (const w of draft.warnings) console.log(w.account_id, w.code, w.message);

// A draft thread may be a single part while it is still being written.
const draftThread = await chirpie.createThread({
  account_id: "uuid",
  posts: [{ text: "Opening line, rest to come" }],
  draft: true,
});

// Ask for them by name: a listing with no status leaves drafts out.
const drafts = await chirpie.listPosts({ status: "draft" });

// Change only the time it remembers, still a draft
await chirpie.updatePost("draft-uuid", {
  schedule_at: "2027-04-02T09:00:00Z",
  draft: true,
});

// Promote it: queue it for a time, or send it now. Never both (400).
await chirpie.updatePost("draft-uuid", { schedule_at: "2027-04-02T09:00:00Z" });
await chirpie.updatePost("draft-uuid", { publish: true });
```

A warning that would really be a refusal carries the same `code` and sentence
the API would answer the send with. Three describe a change rather than a
refusal: `thread_not_native`, `x_link_post_billed`, `schedule_at_in_past`.

Promotion runs every rule a create runs and spends the quota the draft never
spent, so anything refused leaves the draft exactly as it was. The answer is a
new post carrying `promoted_from_draft_id`, or `promoted_from_draft_ids` for a
draft thread, which is promoted whole. `draft: false` is refused with a 400, and
`publish` or `draft` on a post that is not a draft is a 400 too.

### Accounts

```typescript
// List all connected accounts, active and inactive
const accounts = await chirpie.listAccounts();
// Each account: { id, platform, username, display_name, avatar_url, is_active,
//                 inactive_reason }
// inactive_reason: "plan_limit" means connected but not switched on because the
// plan's account limit was full. "token_revoked", "token_expired" and
// "reauth_required" mean the platform stopped accepting the credential, so the
// account has to be connected again and its scheduled posts were cancelled.
// Pause on is_active === false rather than retrying, then read inactive_reason
// to decide between activateAccount() and a fresh connect. A deliberately
// deactivated account carries no inactive_reason at all, so pausing only when
// that field is present would keep posting into a dead account.
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

// A narrower key. Omitting `scopes` copies the calling key's own scopes, so
// a bare name gives full access only when the caller has it. A key can never
// grant a scope it does not itself hold
// (403 insufficient_scope). Vocabulary: posts:read, posts:write,
// accounts:read, accounts:write, analytics:read, comments:read,
// comments:write, media:write, keys:write. There is no keys:read: all three
// key methods need keys:write.
const scoped = await chirpie.createKey({
  name: "Publishing bot",
  scopes: ["posts:write", "media:write"],
});

const keys = await chirpie.listKeys();  // each carries `scopes`; null = full access
await chirpie.revokeKey("key-uuid");
```

### Analytics

```typescript
const metrics = await chirpie.getPostAnalytics("post-uuid");
// { impressions, likes, retweets, replies, quotes, bookmarks, clicks, fetched_at }
// Served from a stored snapshot under an hour old, so polling costs nothing.

// Ask the platform now. Floored at one forced refresh per post every 30
// minutes; past that it throws 429 analytics_refresh_rate_limited with a
// Retry-After, and the stored numbers are still one ordinary call away.
const fresh = await chirpie.getPostAnalytics("post-uuid", { refresh: true });
```

## Types

```typescript
import type {
  ApiPost,                    // Post object (includes `platform` field)
  ApiFirstComment,            // A post's first comment: text, status, comment_id, error
  ApiThread,                  // Thread object returned from API
  ApiAccount,                 // Connected account (platform, username, display_name, avatar_url)
  ApiAnalytics,               // Post metrics
  ApiKeyInfo,                 // API key metadata (prefix only, plus `scopes`)
  ApiScope,                   // One scope: "posts:write", "analytics:read", and so on
  CreateKeyInput,             // createKey({ name, scopes })
  RequestOptions,             // Per-call options: { idempotencyKey }
  CreatePostInput,            // Input for createPost()
  CreateThreadInput,          // Input for createThread()
  UpdatePostInput,            // Input for updatePost()
  DraftWarning,               // One thing that would go wrong if a draft were sent
  DraftPostResponse,          // createPost({ draft: true }): post + warnings
  DraftThreadResponse,        // createThread({ draft: true }): thread + warnings
  FanOutDraftPostResponse,    // A draft post saved for several accounts
  FanOutDraftThreadResponse,  // A draft thread saved for several accounts
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

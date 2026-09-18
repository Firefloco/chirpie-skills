---
name: chirpie-posting
description: Create posts and threads on X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram via the Chirpie API. Covers single posts, multi-post threads, listing, deletion, comments and replies, and analytics.
---

# Chirpie Posting

## Create a Post

```typescript
import { ChirpieClient } from "@chirpie/sdk";

const chirpie = new ChirpieClient({ apiKey: process.env.CHIRPIE_API_KEY! });

const post = await chirpie.createPost({
  account_id: "YOUR_ACCOUNT_ID",
  text: "Hello world!",
});
```

### curl

```bash
curl -X POST https://chirpie.ai/api/v1/posts \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "YOUR_ACCOUNT_ID",
    "text": "Hello world!"
  }'
```

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `account_id` | UUID string | Yes | Connected account ID |
| `text` | string | Yes | X: 1-280 (25,000 on Premium). Bluesky: 300. LinkedIn: 3,000. Threads: 500. Mastodon: 500. Instagram: 2,200. Facebook: 63,206. Telegram: 4,096, or 1,024 when the post carries media. |
| `media_urls` | string[] | No | Public image/video URLs. Max images per post: X 4, Bluesky 4, LinkedIn 4, Threads 1, Mastodon 4, Instagram 10, Facebook 10, Telegram 10. Video: X, Mastodon and Telegram only, 1 per post and never alongside images. Instagram REQUIRES at least one image. Anything a platform cannot take is refused with `400 unsupported_media`, never dropped. |
| `schedule_at` | ISO 8601 | No | Future datetime for scheduling. Must be absolute and carry a timezone (`...Z` or `+02:00`); normalized to UTC |

A missing required field is named: `POST /api/v1/posts {}` returns `account_id and text are required`.

Only these fields are accepted. Any other top-level field returns `400 bad_request`. That includes `scheduled_at` (the field name in the *response*), which is rejected with an `Unknown field 'scheduled_at'` error suggesting `schedule_at`. Never send back a whole post object you read from the API.

### Response (201)

```json
{
  "data": {
    "id": "uuid",
    "account_id": "uuid",
    "text": "Hello world!",
    "media_urls": null,
    "platform": "x",
    "platform_post_id": "1234567890",
    "platform_post_url": "https://x.com/chirpie_ai/status/1234567890",
    "status": "published",
    "scheduled_at": null,
    "published_at": "2026-03-23T10:00:00.000Z",
    "thread_id": null,
    "thread_order": null,
    "error_message": null,
    "created_at": "2026-03-23T10:00:00.000Z"
  }
}
```

`platform_post_url` is the public permalink, or `null` when the platform has none that can be derived. X, Bluesky, Mastodon, LinkedIn, Facebook, and public Telegram channels get a URL; Threads and Instagram are always `null`. Thread responses carry it on each post too.

## Create a Thread

```typescript
const thread = await chirpie.createThread({
  account_id: "YOUR_ACCOUNT_ID",
  posts: [
    { text: "Thread starts here 🧵" },
    { text: "Second tweet in thread" },
    { text: "Final tweet, follow for more!" },
  ],
});
```

### curl

```bash
curl -X POST https://chirpie.ai/api/v1/threads \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "YOUR_ACCOUNT_ID",
    "posts": [
      { "text": "Thread starts here 🧵" },
      { "text": "Second tweet in thread" },
      { "text": "Final tweet, follow for more!" }
    ]
  }'
```

- Min 2 posts, max 25 posts per thread
- Character limits per platform (same as single posts)
- X, Bluesky, Threads, Mastodon, and Telegram support native reply threading.
- LinkedIn, Instagram, and Facebook degrade gracefully: each item is published as a standalone post.
- Thread counts as N posts against your monthly quota

## List Posts

```typescript
const posts = await chirpie.listPosts({
  status: "published",  // Optional: draft|scheduled|publishing|published|failed|deleted
  account_id: "uuid",   // Optional
  limit: 20,            // Max 100
  offset: 0,
});
```

## Get a Post

```typescript
const post = await chirpie.getPost("post-uuid");
```

## Delete a Post

```typescript
const result = await chirpie.deletePost("post-uuid");
// Removes it from the platform first, and only then from Chirpie. If the platform
// refuses, nothing changes and the call throws upstream_error: retry the same call.
// Instagram and TikTok have no delete API, so posts there stay live.
```

## Comments and Replies

For a post Chirpie published, list the comments it received, reply to one, hide one, or delete one.

```typescript
// List. Newest first. Page with next_cursor until it comes back null.
const { comments, next_cursor, capabilities, sync } = await chirpie.listComments(
  "post-uuid",
  { limit: 25, include_hidden: false }
);

// Reply. Published to the platform, so it counts as ONE POST against the monthly quota.
await chirpie.replyToComment("post-uuid", "comment-uuid", "Thanks, that is on the roadmap.");

// Hide and unhide. Facebook, Instagram and Threads only.
await chirpie.hideComment("post-uuid", "comment-uuid");
await chirpie.unhideComment("post-uuid", "comment-uuid");

// Delete.
await chirpie.deleteComment("post-uuid", "comment-uuid");
```

### curl

```bash
curl "https://chirpie.ai/api/v1/posts/POST_ID/comments?limit=25" \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY"

curl -X POST https://chirpie.ai/api/v1/posts/POST_ID/comments/COMMENT_ID/reply \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "text": "Thanks, that is on the roadmap." }'
```

### What each platform allows

| Platform | List | Reply | Hide | Delete |
|----------|------|-------|------|--------|
| X/Twitter | Yes | Yes | No | Your own replies |
| Bluesky | Yes | Yes | No | Your own replies |
| Mastodon | Yes | Yes | No | Your own replies |
| LinkedIn (Page) | Yes | Yes | No | Your own comments |
| LinkedIn (profile) | No | No | No | No |
| Facebook _(Coming Soon)_ | Yes | Yes | Yes | Any comment |
| Instagram _(Coming Soon)_ | Yes | Yes | Yes | Any comment |
| Threads _(Coming Soon)_ | Yes | Yes | Yes | Your own replies |
| Telegram | No | No | No | No |

**Never hard-code that table.** Every listing carries `meta.capabilities` for the post you asked about: `{ reply, hide, delete_own, delete_any }`. Check it before offering a control. An action the platform does not have returns `501 comment_action_unsupported` rather than silently doing nothing.

### Freshness

A listing answers from comments Chirpie has stored, refreshed on a schedule. `meta.sync.status` says how fresh they are: `ok`, or one of `unsupported`, `permission`, `rate_limited`, `budget`, `plan`, `error`. Anything but `ok` carries a ready-worded sentence in `meta.sync.reason` that can be shown to a user as is. Pass `sync=true` to ask for a refresh now, and `sync=false` to skip one. A refused refresh is still a `200` with the stored comments.

`409 comment_permission_required` means the account has to be reconnected at https://chirpie.ai/dashboard/accounts before its comments can be read or managed. Posting and analytics keep working meanwhile.

### Cost

- Listing and hiding cost nothing.
- **A reply is a post**, so it uses one from the monthly quota, and an X reply containing a link carries the same X link-post charge a normal X post does.
- Refreshing is metered per plan: 200 syncs/mo on Free, 1,000 on Agent, 5,000 on Starter, 25,000 on Pro. X is metered a second time per reply returned, and is not included on Free.

## Get Analytics

```typescript
const analytics = await chirpie.getPostAnalytics("post-uuid");
// Returns: post_id, platform, impressions, likes, retweets, replies, quotes,
// bookmarks, clicks, fetched_at. Metrics a platform does not expose come back as 0.
// Cached for 1 hour. Telegram exposes no metrics API, so it returns 502.
```

## Error Handling

```typescript
import { ChirpieApiError } from "@chirpie/sdk";

try {
  await chirpie.createPost({ ... });
} catch (err) {
  if (err instanceof ChirpieApiError) {
    switch (err.status) {
      case 400: // Invalid request (check err.message for details)
      case 401: // Invalid API key
      case 404: // Account not found or inactive
      case 429: // Rate limited (monthly quota or burst), or account_limit_reached
      case 502: // Platform API error (temporary, retry)
      case 503: // Media could not be stored (temporary, retry; nothing was published)
    }
  }
}
```

## Post Status Flow

```
immediate:  → published | failed
scheduled:  → scheduled → publishing → published | failed
deleted:    → deleted (also removed from platform if published, except Instagram)
```

Failed posts: check `error_message` field for details.

## Rate Limit Headers

Every authenticated `/api/v1/*` response carries `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (unix seconds). A `429` from the burst limiter also carries `Retry-After` (seconds): sleep that long and retry once. A `429` with no `Retry-After` is a quota or account-limit refusal, so do not retry it.

---
name: chirpie-posting
description: Create posts and threads on X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, Telegram, Pinterest, TikTok, YouTube, and Google Business Profile via the Chirpie API. Covers single posts, multi-post threads, listing, deletion, and analytics.
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
| `text` | string | Yes | X: 1-280 (25K Premium). Bluesky: 300. LinkedIn: 3K. Threads: 500. Mastodon: 500. Instagram: 2,200. Facebook: 63,206. Telegram: 4,096. Pinterest: 500. TikTok: 2,200. YouTube: 5K. Google Business: 1,500. |
| `media_urls` | string[] | No | Public image/video URLs. Limits vary by platform. Instagram, Pinterest, TikTok, and YouTube REQUIRE media. |
| `schedule_at` | ISO 8601 | No | Future datetime for scheduling. Must carry a timezone (`...Z` or `+02:00`); normalized to UTC |

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
- LinkedIn, Instagram, Facebook, Pinterest, TikTok, YouTube, and Google Business Profile degrade gracefully: each item is published as a standalone post.
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
// Also deletes from the platform if published (except Instagram and TikTok, which have no delete API)
```

## Get Analytics

```typescript
const analytics = await chirpie.getPostAnalytics("post-uuid");
// Returns: impressions, likes, retweets, replies, quotes, bookmarks, clicks
// Cached for 1 hour
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
      case 429: // Rate limited (monthly quota or burst)
      case 502: // Platform API error (temporary, retry)
    }
  }
}
```

## Post Status Flow

```
immediate:  → published | failed
scheduled:  → scheduled → publishing → published | failed
deleted:    → deleted (also removed from platform if published, except Instagram and TikTok)
```

Failed posts: check `error_message` field for details.

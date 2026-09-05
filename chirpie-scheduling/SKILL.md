---
name: chirpie-scheduling
description: Schedule posts and threads for future publishing with Chirpie. Covers timing, retries, cancellation, and scheduled post limits.
---

# Chirpie Scheduling

## Schedule a Post

Add `schedule_at` (ISO 8601, UTC, must be in the future) to any create request:

```typescript
const post = await chirpie.createPost({
  account_id: "YOUR_ACCOUNT_ID",
  text: "This posts tomorrow at noon!",
  schedule_at: "2026-03-24T12:00:00Z",
});
// post.status === "scheduled"
```

### curl

```bash
curl -X POST https://chirpie.ai/api/v1/posts \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "YOUR_ACCOUNT_ID",
    "text": "This posts tomorrow at noon!",
    "schedule_at": "2026-03-24T12:00:00Z"
  }'
```

## Schedule a Thread

All posts in a thread share the same `schedule_at`:

```typescript
const thread = await chirpie.createThread({
  account_id: "YOUR_ACCOUNT_ID",
  posts: [
    { text: "Scheduled thread post 1" },
    { text: "Scheduled thread post 2" },
  ],
  schedule_at: "2026-03-24T12:00:00Z",
});
```

Threads publish atomically — if any post fails, the entire thread retries.

## How It Works

1. Posts are stored with `status: "scheduled"`
2. A cron job runs **every 5 minutes** checking for posts due to publish
3. Posts publish within ~5 minutes of their `schedule_at` time
4. All times are **UTC**

## Cancel a Scheduled Post

Delete it before the scheduled time:

```typescript
await chirpie.deletePost("post-uuid");
```

## Retry Behavior

- Failed posts retry up to **3 times** with 5-minute delays
- After 3 failures → `status: "failed"` with `error_message`
- Threads: if any post fails, the whole thread is rescheduled

## Scheduled Post Limits

Scheduled posts have a separate monthly quota:

| Plan | Scheduled/mo |
|------|-------------|
| Free | 25 |
| Agent | 150 |
| Starter | 500 |
| Pro | 2,500 |
| Scale | 12,500 |
| Enterprise | Unlimited |

Both `posts` and `scheduled` limits are checked when creating scheduled content. A scheduled thread of 5 posts counts as 5 against both quotas.

## Common Pitfalls

1. **`schedule_at` must be in the future.** Past datetimes return 400.
2. **Times are UTC.** Convert from local time before sending.
3. **Thread atomicity.** You can't cancel individual posts in a scheduled thread — delete any one post and the whole thread is canceled.
4. **Platform rate limits.** If any platform rate-limits your account, scheduled posts will retry automatically.

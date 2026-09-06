---
name: chirpie-scheduling
description: Schedule posts and threads for future publishing with Chirpie. Covers timing, retries, cancellation, and scheduled post limits.
---

# Chirpie Scheduling

## Schedule a Post

Add `schedule_at` (ISO 8601, must be in the future and carry a timezone such as `...Z`, `+02:00`, or `-05:00`, and normalized to UTC) to any create request:

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

Threads publish atomically: if any post fails, the entire thread retries.

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

1. **The field is `schedule_at`, not `scheduled_at`.** `scheduled_at` is the field name in the *response*. Sending it returns a `400` error naming the unknown field `scheduled_at` and suggesting `schedule_at`; the post is not published. Only `account_id`, `text`, `media_urls` and `schedule_at` (or `account_id`, `posts`, `schedule_at` for a thread) are accepted, so do not send back a whole post object you read from the API.
2. **`schedule_at` must be in the future, absolute, and carry a timezone.** Each failure has its own 400 message: no timezone ("schedule_at is missing a timezone, so the instant it names is ambiguous. Add 'Z' for UTC or an offset like '+02:00'."), a relative offset such as `+30m` ("schedule_at must be an absolute ISO 8601 timestamp, not a relative offset like '+30m'."), anything else malformed ("schedule_at must be an ISO 8601 timestamp."). Past datetimes also return 400.
3. **Times are UTC.** Convert from local time before sending.
4. **Thread atomicity.** You can't cancel individual posts in a scheduled thread: delete any one post and the whole thread is canceled.
5. **Platform rate limits.** If any platform rate-limits your account, scheduled posts will retry automatically.

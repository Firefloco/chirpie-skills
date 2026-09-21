---
name: chirpie-scheduling
description: Schedule posts and threads for future publishing with Chirpie. Covers timing, retries, cancellation, drafts and promoting them, and scheduled post limits.
---

# Chirpie Scheduling

## Schedule a Post

Add `schedule_at` (ISO 8601, in the future) to any create request. It may be an absolute instant carrying a timezone (`...Z`, `+02:00`, `-05:00`), normalized to UTC, or a local time with no offset read in a `timezone` field or the one saved on the account. See "Timezones" below.

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
4. Everything Chirpie returns (`scheduled_at`, `published_at`) is **UTC**

## Timezones

There are three ways to name a time, and only the first one existed before.

| What you send | How it is read |
|---|---|
| `"schedule_at": "2026-11-01T13:30:00Z"` or any offset | An absolute instant. `timezone` is not consulted, and nothing about this changed |
| `"schedule_at": "2026-11-01T09:30:00"` plus `"timezone": "America/New_York"` | 09:30 wall-clock in that zone on that date, using the offset actually in force then |
| `"schedule_at": "2026-11-01T09:30:00"` with a timezone saved on the account | The same, with the saved zone standing in |

A local time with no zone on the request and none saved is refused with a `400`.

`timezone` is an IANA name (`America/New_York`, `Europe/Berlin`), accepted on `POST /api/v1/posts`, `POST /api/v1/threads` and `PATCH /api/v1/posts/:id`. **A fixed offset like `+02:00` is NOT accepted there**, on purpose: an offset is only right until the clocks change. Put an offset on `schedule_at` itself if that is what you want.

**Why it matters.** 09:30 on 1 October 2026 in `America/New_York` is `13:30Z` (UTC-4, daylight time). 09:30 on 1 November 2026 in the same zone is `14:30Z` (UTC-5, standard time), because the clocks go back that morning. A client that computes today's offset and sends `09:30:00-04:00` for the November post publishes it an hour early. Send the zone name and let Chirpie resolve the offset for the date named.

The customer sets a default timezone in the dashboard under Settings. A `timezone` on the request always wins over it. Everything Chirpie returns (`scheduled_at`, `published_at`) is still UTC.

Client surfaces: SDK `timezone` on `CreatePostInput`, `CreateThreadInput` and `UpdatePostInput`; CLI `--timezone <iana>` on `post`, `thread` and `posts update`; MCP `timezone` on `chirpie_post`, `chirpie_thread` and `chirpie_update_post`; n8n a **Timezone** option.

## Idempotency

A scheduled create is worth an `Idempotency-Key` header: a retry after a timeout would otherwise queue the same post twice. The same key with the same request replays the first answer for 24 hours (with `Idempotent-Replay: true`); the same key with a different request is `422 idempotency_key_reused`; a retry arriving while the first is still running is `409 idempotency_in_progress`, which does not wait, so retry once more to collect the replay. See the chirpie-posting skill for the full rules.

## Minimum Spacing

Two scheduled posts on the **same account** must be at least 5 minutes apart. A closer
time returns `400 bad_request`:
`Scheduled posts must be at least 5 minutes apart for the same account.`

## Cancel a Scheduled Post

Delete it before the scheduled time:

```typescript
await chirpie.deletePost("post-uuid");
```

## Disconnecting or Deactivating an Account Cancels Its Queue

A disconnected or deactivated account cannot publish, so **every scheduled post queued
against it is canceled**, and both quota counters (posts and scheduled posts) are returned.

```typescript
const accounts = await chirpie.listAccounts();
const account = accounts.find((a) => a.id === id)!;
// How many scheduled posts deactivating would cancel.
console.log(account.scheduled_posts);

const updated = await chirpie.deactivateAccount(id);
console.log(updated.canceled_posts); // how many actually went

// Disconnecting cancels the queue the same way, and also removes the stored
// credential, so connecting the account again means authorizing it on the platform.
const gone = await chirpie.disconnectAccount(id);
console.log(gone.canceled_posts);
```

Reactivating (or reconnecting) does **not** restore them, so schedule them afresh. Check
`scheduled_posts` and confirm with the user before deactivating or disconnecting an account
that has any.

## Retry Behavior

- Failed posts retry up to **3 times** with 5-minute delays
- After 3 failures → `status: "failed"` with `error_message`; `retry_count` says how many attempts were made
- Threads: if any post fails, the whole thread is rescheduled
- If the **connection** dies, retrying cannot help: the account is switched off with an `inactive_reason`, its whole scheduled queue is cancelled (`error_message: "connection_lost"`) and the quota returned. Pause and tell the user to reconnect: https://chirpie.ai/docs/connection-dies

## Changing a Scheduled Post

`PATCH /api/v1/posts/:id` edits a post that has not gone out yet. `text`, the media (`media`, `media_ids` or `media_urls`) and `schedule_at` are all optional, and only what is sent changes. On a draft it also takes `draft` and `publish`: see "Drafts" below.

```bash
# Fix the words, keep the time
curl -X PATCH https://chirpie.ai/api/v1/posts/POST_ID \
  -H "Authorization: Bearer $CHIRPIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"text": "Now with the typo fixed"}'

# Move it, keep the words
curl -X PATCH https://chirpie.ai/api/v1/posts/POST_ID \
  -H "Authorization: Bearer $CHIRPIE_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"schedule_at": "2027-04-02T09:00:00Z"}'
```

**Leaving `schedule_at` out keeps the time the post already has.** The call never publishes a queued post. Promoting a draft is the one thing it does publish, and only when the request asks for it.

A new time obeys the same rules as the original: in the future, and at least 5 minutes from any other scheduled post on the same account (the post being moved does not count against itself). Rescheduling one post of a thread moves every part of it, and `rescheduled_post_ids` names them.

Only a post that has not published yet can be edited, a scheduled post or a draft: `409 post_not_editable` otherwise, which includes a draft somebody else already promoted. Editing does not count against the monthly quota.

## Drafts

`draft: true` on a create saves the content and sends nothing. A draft never publishes on its own and never enters the scheduler queue, so it costs neither quota until it is promoted. A `schedule_at` on a draft is only the time the customer has in mind, and need not be in the future.

```typescript
const draft = await chirpie.createPost({
  account_id: "YOUR_ACCOUNT_ID",
  text: "Half an idea. Finish it later.",
  schedule_at: "2026-03-24T12:00:00Z",
  draft: true,
});
// draft.status === "draft"; draft.warnings says what would go wrong if it were sent

const drafts = await chirpie.listPosts({ status: "draft" });

// Change only the time it remembers
await chirpie.updatePost(draft.id, { schedule_at: "2027-03-25T12:00:00Z", draft: true });

// Promote it: queue it, or send it now. Promoting needs a future time.
await chirpie.updatePost(draft.id, { schedule_at: "2027-03-25T12:00:00Z" });
await chirpie.updatePost(draft.id, { publish: true });
```

Rules to hold on to:

- `publish: true` and `schedule_at` are never valid together (400). `draft: false` is refused with a 400 saying to send one of them. On a post that is not a draft, both `publish` and `draft` are 400.
- Promotion runs every rule a create runs, the 5 minute spacing and the two-post thread minimum included, and spends the quota the draft never spent. Anything refused leaves the draft exactly as it was.
- The answer is a **new** post carrying `promoted_from_draft_id`, or `promoted_from_draft_ids` for a draft thread, which is promoted whole. The draft's own id is gone.
- A draft whose remembered time passes stays a draft. Nothing publishes late and nothing is cancelled.

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

1. **The field is `schedule_at`, not `scheduled_at`.** `scheduled_at` is the field name in the *response*. Sending it returns a `400` error naming the unknown field `scheduled_at` and suggesting `schedule_at`; the post is not published. Only `account_id`, `account_ids`, `account_configurations`, `text`, `media`, `media_ids`, `media_urls`, `first_comment`, `schedule_at`, `timezone` and `draft` (or `account_id`, `account_ids`, `account_configurations`, `posts`, `first_comment`, `schedule_at`, `timezone`, `draft` for a thread) are accepted, so do not send back a whole post object you read from the API.
2. **`schedule_at` must be in the future and resolvable to an instant**, except on a draft, where it is only a remembered time. Each failure has its own 400 message: a local time with no `timezone` on the request and none saved on the account ("schedule_at is missing a timezone, so the instant it names is ambiguous. Add 'Z' for UTC or an offset like '+02:00', send a timezone field such as 'America/New_York', or save a timezone on your account."), a `timezone` that is not an IANA name ("timezone must be an IANA timezone name such as 'America/New_York' or 'Europe/Berlin', not '+02:00'."), a relative offset such as `+30m` ("schedule_at must be an absolute ISO 8601 timestamp, not a relative offset like '+30m'."), a local time that is the right shape but not a real moment, such as `2026-02-30T09:30` or `2026-01-01T25:00` ("schedule_at is not a real date and time: '...'. Check the day of the month and the hour."), anything else malformed ("schedule_at must be an ISO 8601 timestamp."). Past datetimes also return 400.
3. **Do not convert local time to UTC yourself.** Send the local time with a `timezone` such as `America/New_York`, or save one on the account. Computing an offset from today is what goes an hour wrong across a clock change. Everything Chirpie returns is UTC.
4. **The two hours a year that are not simple.** A local time on the night the clocks change is resolved the way every scheduler resolves it, and never earlier than you named. A wall time that happens twice (the clocks go back) takes the **first** occurrence; one that does not exist (the clocks go forward) is shifted forward by the length of the gap, so `02:30` in a one-hour gap publishes at `03:30` local. Correct on zones that move by half an hour too.
5. **Thread atomicity.** You can't cancel individual posts in a scheduled thread: delete any one post and the whole thread is canceled.
6. **Platform rate limits.** If any platform rate-limits your account, scheduled posts will retry automatically.

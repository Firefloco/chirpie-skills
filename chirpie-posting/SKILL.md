---
name: chirpie-posting
description: Create posts and threads on X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram via the Chirpie API. Covers single posts, posting to several accounts at once with per-account overrides, multi-post threads, the first comment, drafts, listing, deletion, comments and replies, and analytics.
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
  -H "Idempotency-Key: 8f1c0e0c-6d51-4d73-9f4e-6b2a0a2d3f11" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "YOUR_ACCOUNT_ID",
    "text": "Hello world!"
  }'
```

### Idempotency: never publish twice on a retry

`POST /api/v1/posts`, `POST /api/v1/threads`, `POST /api/v1/media`, `POST /api/v1/posts/:id/first-comment` and `POST /api/v1/posts/:id/comments/:commentId/reply` accept an `Idempotency-Key` header. Send one on anything you might retry.

- Same key, same request: the original response is replayed, carrying `Idempotent-Replay: true`. Nothing publishes again and no quota is spent.
- Same key, a different method, path or body: `422 idempotency_key_reused`. Use a fresh key per distinct request.
- Same key while the first call is still running: `409 idempotency_in_progress`. It does not wait. Retry once more to collect the replay.
- Anything under 500 is stored and replayed, errors included. A 5xx releases the key, so a retry is a real retry. The exception is `502 thread_rollback_incomplete`, which means parts of a thread are still live on the platform: that one is kept and replayed, because the retry would otherwise publish them a second time.
- Keys last 24 hours and may be up to 255 characters. A blank or longer value is refused with `400 idempotency_key_invalid` rather than ignored: a caller who sends a key is asking for deduplication, so answering `201` without providing any would be worse.

The SDK's `createPost()` and `createThread()` generate a fresh key per call, which protects the one HTTP request it rides on. **Calling the method again is a new call and gets a new key**, so to make your own retry replay rather than republish, pass the same key each time: `{ idempotencyKey }` as the second argument, or `{ idempotencyKey: null }` to send none. `uploadMedia(input, options)`, `retryFirstComment(id, options)` and `replyToComment(postId, commentId, text, options)` take the same options object and generate nothing.

### Uploading a file

`POST /api/v1/media` takes a `multipart/form-data` body with the file in a part named `file`, or a JSON body naming a public `url`, and answers `201` with `{ "data": { "id", "url", "mime_type", "media_type", "bytes", "expires_at", ... } }`. Attach the id with `media: [{ "id": "...", "alt": "..." }]` or `media_ids: ["..."]`.

The file type is read from the file's own first bytes, never from its name. Uploads are limited to 4 MB as a `file` part, or 3 MB of file as base64 `data`, which bounds the request rather than the post; a larger file is attached by URL instead, which has no such limit. An id is good for 7 days, and a post keeps its own copy of the bytes, so it publishes as written whatever happens to the id. A post still waiting to publish also keeps its id alive, so it stays editable; a post reads back as `media: [{ url, alt, media_id }]`, and an edit should send an uploaded item back as `{ "id": media_id }` rather than by its URL.

### Request Body

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `account_id` | UUID string | Yes, unless `account_ids` is given | Connected account ID |
| `account_ids` | UUID string[] | Yes, unless `account_id` is given | 1 to 25 connected accounts, published to in one request. Duplicates are refused. Naming it selects the fan-out response, even for one account. See "Post to several accounts at once" below |
| `account_configurations` | object | No | Per-account overrides keyed by account ID. Only with `account_ids`, and every key must be in it. Each value takes `text`, `first_comment` (`""` publishes that account with none), `configuration` (replacing the whole block for that account), and one of `media`, `media_ids`, `media_urls` |
| `text` | string | Yes, unless `draft` is true | X: 1-280 (25,000 on Premium). Bluesky: 300. LinkedIn: 3,000. Threads: 500. Mastodon: 500. Instagram: 2,200. Facebook: 63,206. Telegram: 4,096, or 1,024 when the post carries media. |
| `media` | object[] | No | Uploaded files and public links, each `{ id? , url?, alt? }`. `id` comes from `POST /api/v1/media`; `alt` describes the item for screen readers and is sent to X, Bluesky, LinkedIn, Mastodon, Instagram and Facebook. Use one of `media`, `media_ids` and `media_urls`, not several. |
| `media_ids` | string[] | No | Ids from `POST /api/v1/media`, when no alt text is needed. |
| `media_urls` | string[] | No | Public image/video URLs. Max images per post: X 4, Bluesky 4, LinkedIn 4, Threads 1, Mastodon 4, Instagram 10, Facebook 10, Telegram 10. Video: X, Mastodon and Telegram, 1 per post and never alongside images, plus Instagram and Facebook where the placement takes one (an Instagram story or Page story takes one image or video, an Instagram reel takes one video and no images). Instagram REQUIRES media on every post. Anything a platform cannot take is refused with `400 unsupported_media`, never dropped. |
| `first_comment` | string | No | A comment published under the post the moment it goes out. X, Threads, Instagram and Facebook only: anywhere else the request is refused with `400 first_comment_unsupported`, never dropped. Counts as one post against the monthly quota. See "First Comment" below |
| `configuration` | object | No | Per-platform publishing options, keyed by platform. Instagram takes `feed` (the default), `story` or `reel`; a Facebook Page takes `feed` or `story`. Anything the platform or the placement does not carry is refused with `400 configuration_unsupported`, never dropped. See "Publishing Options" below |
| `schedule_at` | ISO 8601 | No | Future datetime for scheduling. Either absolute, carrying a timezone (`...Z` or `+02:00`), normalized to UTC, or a local time with no offset (`2026-11-01T09:30:00`) read in `timezone` or the timezone saved on the account. A local time with neither is refused. On a draft it is only the time to remember, and may be any time at all |
| `timezone` | IANA name | No | The zone a `schedule_at` with no offset is read in, such as `America/New_York`. Daylight saving is worked out for the date named, which a client computing today's offset gets wrong across a clock change. A fixed offset (`+02:00`) is NOT accepted here: put it on `schedule_at` instead. Also accepted on `POST /api/v1/threads` and `PATCH /api/v1/posts/:id` |
| `draft` | boolean | No | Save without sending. Nothing reaches the platform and nothing counts against the quota. See "Drafts" below |

A missing required field is named: `POST /api/v1/posts {}` returns `account_id and text are required`.

Only these fields are accepted. Any other top-level field returns `400 bad_request`. That includes `scheduled_at` (the field name in the *response*), which is rejected with an `Unknown field 'scheduled_at'` error suggesting `schedule_at`, and `group_id`, which every returned post carries but no request may send. Never send back a whole post object you read from the API.

### First Comment

`first_comment` publishes one comment under the post the moment it goes out: the "link in the first comment" pattern. It is posted by the same account, recorded as one of the customer's own comments, and appears in the post's comment thread with `own: true`.

```bash
curl -X POST https://chirpie.ai/api/v1/posts \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_id": "YOUR_ACCOUNT_ID",
    "text": "We rebuilt scheduling this week.",
    "first_comment": "Full write-up: https://example.com/blog/scheduling"
  }'
```

- **Platforms**: X, Threads, Instagram and Facebook. Anywhere else the request is refused with `400 first_comment_unsupported`, naming the platform and the four that work. It is never silently dropped.
- **Length**: the account's own post limit (280 standard X, 25,000 X Premium, 500 Threads, 2,200 Instagram, 63,206 Facebook).
- **Quota**: a first comment is a post on the platform, so it counts as one post against the monthly quota, exactly as a comment reply does.
- **X links**: a first comment containing a link carries the same $0.25 charge on paid plans as a link post, and is refused on the Free plan with `402 x_link_posts_require_paid_plan` before the post is published, so nothing goes out. X accounts using the customer's own X API keys are exempt.
- **On a thread**: `POST /api/v1/threads` takes `first_comment` too. One comment for the whole thread, published under the **last** part and reported on that part.
- **On a fan-out**: a top-level `first_comment` applies to every account. An `account_configurations` entry with no `first_comment` inherits it, one naming a `first_comment` replaces it for that account, and one setting `"first_comment": ""` publishes that account with none. The empty string is how a single request sends a first comment to the accounts that take one while an account whose platform has none still publishes the post; without it that account refuses the whole request with `first_comment_unsupported`.
- **On a draft**: stored with the draft and carried into the post when it is promoted. A draft whose account cannot take one comes back with a `warnings` entry coded `first_comment_unsupported`.
- **On a scheduled post**: `status` stays `pending` until the post publishes, and both go out at the scheduled time, the post first.

Every post payload carries `first_comment`, either `null` or:

```json
{ "text": "Full write-up: https://example.com", "status": "posted", "comment_id": "uuid", "error": null }
```

`status` is `pending`, `posted` or `failed`. `pending` means the post has not published yet, or the comment is on its way. `comment_id` is the comment's id in the post's comment thread, so it can be read, hidden or deleted through the comments API like any other comment.

**A failed first comment never fails its post.** The post publishes, `first_comment.status` is `failed`, and `error` carries a worded reason.

### Retry a First Comment

```bash
curl -X POST https://chirpie.ai/api/v1/posts/POST_ID/first-comment \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY"
```

No request body: it re-sends the text the post already carries, and there is no way to change that text once the post is out, because `PATCH /api/v1/posts/:id` refuses a published post with `409 post_not_editable`. Write or change a first comment while the post is still a draft or still queued, with `PATCH /api/v1/posts/:id` (send `""` to remove it). The answer is `200` with the post, `first_comment.status` now `posted`. A retry that works costs one post from the monthly quota, like the first attempt.

Refusals: `404 first_comment_not_found` (the post has none), `409 first_comment_not_retryable` (already posted, the post has not published yet, or another attempt is in flight), and `502 first_comment_failed` when the platform refuses the retry too.

### Publishing Options

`configuration` says how a platform publishes the post, keyed by platform slug. Instagram and Facebook are the two platforms that offer a choice; every other platform publishes one way and takes no block.

```json
{
  "account_id": "YOUR_ACCOUNT_ID",
  "text": "Behind the scenes of this week.",
  "media_urls": ["https://example.com/clip.mp4"],
  "configuration": { "instagram": { "placement": "reel", "share_to_feed": true } }
}
```

Connecting a new Instagram account or Facebook Page is coming soon. Accounts already connected publish and schedule with these options as described here.

| Platform | Placements | Options that placement carries |
|---|---|---|
| Instagram | `feed` (default) | `collaborators` (up to 3 usernames, no `@`), `user_tags` (up to 20 `{ username, x, y }`, both coordinates required, single-image posts only) |
| Instagram | `story` | `user_tags` (coordinates optional: both or neither) |
| Instagram | `reel` | `collaborators`, `user_tags` (no coordinates), `cover` (an uploaded image id), `video_cover_timestamp_ms`, `share_to_feed`, `trial_reel: { graduation: "manual" \| "performance" }` |
| Facebook | `feed` (default) | `link` (a URL rendered as a preview) |
| Facebook | `story` | nothing beyond `placement` |

Media per placement: an Instagram feed post takes 1 to 10 images (JPEG/PNG, 8 MB each) and no video; a story takes exactly one image or video (MP4/MOV, 100 MB); a reel takes exactly one video (300 MB) and no images. A Facebook feed post takes up to 10 images (JPEG/PNG/GIF, 10 MB each) and no video; a Page story takes exactly one image or video (100 MB).

Rules that catch people out:

- **Nothing is dropped quietly.** A block keyed on a platform that takes none, on a platform no account in the request uses, or a field the placement does not carry, is refused with `400 configuration_unsupported` naming the field. Nothing publishes.
- **A story carries no caption and no first comment.** Send empty text and leave `first_comment` out, on both platforms.
- **A story or a reel is a single post**, so `POST /api/v1/threads` refuses either placement.
- **`cover` and `video_cover_timestamp_ms` are alternatives**, never both.
- **A fan-out shares one block per platform.** An `account_configurations` entry replaces the whole block for that account rather than merging into it.
- **What Chirpie cannot check** is aspect ratio, frame rate and duration, because it never decodes a video. Meta refuses those at publish time and the reason comes back as `502 upstream_error`. A story video runs 3 to 60 seconds on Instagram and up to 60 on a Facebook Page; a reel runs 3 seconds to 15 minutes; both are shown at 9:16.
- **Delete truth follows the placement.** A published Instagram post cannot be deleted at all, and Facebook publishes a delete for a Page feed post but none for a Page story, so a published story answers `501 delete_unsupported`. A Page story expires on its own 24 hours after it was posted.
- **On a PATCH**, `configuration` replaces the post's options and `{}` puts it back to a plain feed post. The media is checked again against the new placement.

Full reference: https://chirpie.ai/docs/platforms/instagram and https://chirpie.ai/docs/platforms/facebook

### Editing a post that has not gone out

`PATCH /api/v1/posts/:id` changes `text`, the media (`media`, `media_ids` or `media_urls`), `first_comment` (an empty string removes it), `configuration` (an empty object puts the post back to a plain feed post) or `schedule_at` on a post still waiting to publish, and on a draft also takes `draft` and `publish`. All are optional, and it accepts only those: `account_id` is not among them, because an edit never moves a post to another account.

**Leaving `schedule_at` out keeps the time the post already has.** The call never publishes a queued post. A draft is the one thing it can send, and only when asked: see "Drafts" below.

Only a post that has not published yet can be edited, which means a scheduled post or a draft. A `publishing`, `published`, `failed` or `deleted` post answers `409 post_not_editable`, and so does a draft that has already been promoted. Editing does not count against the monthly quota. Rescheduling one post of a thread moves every part of it, and `rescheduled_post_ids` names them.

### Response (201)

```json
{
  "data": {
    "id": "uuid",
    "account_id": "uuid",
    "text": "Hello world!",
    "media_urls": null,
    "first_comment": null,
    "platform": "x",
    "platform_post_id": "1234567890",
    "platform_post_url": "https://x.com/chirpie_ai/status/1234567890",
    "status": "published",
    "scheduled_at": null,
    "published_at": "2026-03-23T10:00:00.000Z",
    "thread_id": null,
    "thread_order": null,
    "group_id": null,
    "error_message": null,
    "retry_count": 0,
    "created_at": "2026-03-23T10:00:00.000Z"
  }
}
```

`platform_post_url` is the public permalink, or `null` when the platform has none that can be derived. X, Bluesky, Mastodon, LinkedIn, Facebook, and public Telegram channels get a URL; Threads and Instagram are always `null`. Thread responses carry it on each post too.

## Post to Several Accounts at Once

Name `account_ids` instead of `account_id` on `POST /api/v1/posts` or `POST /api/v1/threads`. One request publishes to up to 25 connected accounts and answers with a `group_id` plus one result per account, in the order the accounts were named.

```typescript
const { group_id, results } = await chirpie.createPost({
  account_ids: ["uuid-a", "uuid-b"],
  text: "Shared text",
  account_configurations: {
    "uuid-b": { text: "Text for just this account" },
  },
});

for (const r of results) {
  if (!r.success) console.error(r.account_id, r.error.code, r.error.message);
}
```

```bash
curl -X POST https://chirpie.ai/api/v1/posts \
  -H "Authorization: Bearer chirpie_sk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "account_ids": ["ACCOUNT_A", "ACCOUNT_B"],
    "text": "Shared text",
    "account_configurations": {
      "ACCOUNT_B": { "text": "Text for just this account" }
    }
  }'
```

### Overrides

- A field left out of an override inherits the request's own value. `text` at the top level is still required: it is what an account without an override publishes.
- **Media replaces, it never merges.** An account whose override names any media spelling (`media`, `media_ids` or `media_urls`, one at a time) replaces the shared media outright for that account. `"media": []` is how one account publishes with no media while the others carry it.
- On `POST /api/v1/threads` the override field is `posts`, and it replaces the whole array for that account, still 2 to 25 parts.

### Response

`201 Created` when every account succeeded, `207 Multi-Status` when at least one did not. Both carry the same body, so **branch on `results[].success`, not on the status code**.

```json
{
  "data": {
    "group_id": "uuid",
    "results": [
      {
        "account_id": "uuid",
        "platform": "x",
        "success": true,
        "post_id": "uuid",
        "platform_post_url": "https://x.com/...",
        "status": "published",
        "post": { "...": "the usual post object" },
        "error": null
      },
      {
        "account_id": "uuid",
        "platform": "bluesky",
        "success": false,
        "post_id": null,
        "platform_post_url": null,
        "status": null,
        "post": null,
        "error": { "code": "upstream_error", "message": "Bluesky API error: ..." }
      }
    ]
  }
}
```

Thread results carry `thread_id` and `thread` where post results carry `post_id` and `post`, and `platform_post_url` is the first part's permalink.

### What is refused whole, and what fails per account

Before anything publishes, every account is resolved and its body, with that account's overrides applied, is run through the same rules a single-account create runs: character limit, media caps, per-platform media rules, alt-text limits, the platforms that require media, whether the platform takes a first comment (give an account that does not `"first_comment": ""`), the publishing options the placement carries, and the X link-post rule. Any of those refuses the **whole request** with its usual status and code, and the message is prefixed `Account <id>: `. Nothing publishes and no quota is taken.

Only the platform call itself fails per account. Those land in `results[].error` and the accounts that worked stay published.

A duplicate id in `account_ids`, or an `account_configurations` key that is not in `account_ids`, is `400 bad_request`.

### Quota

One reservation for the whole group: one unit per account for a post, one per part per account for a thread. If the group does not fit the plan the request is `429 usage_limit_exceeded` and nothing publishes. Each account that fails to publish gives back exactly its own share, once.

### Scheduling and reading a group back

`schedule_at` applies to the whole group, so every account is queued for the same time and every scheduled post shares the group id.

Every post payload carries `group_id`: the fan-out it belongs to, or `null` for a post created with a single `account_id`. Read a whole group back with `GET /api/v1/posts?group_id=...` (`listPosts({ group_id })`). It must be a UUID; anything else is `400`.

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
- Thread counts as N posts against your monthly quota, N + 1 when it carries a first comment
- `first_comment` is one comment for the whole thread, published under the **last** part and reported on that part. See "First Comment" above
- `configuration` applies to every part alike, and a thread publishes to the feed: an Instagram story or reel, and a Facebook Page story, are each a single post, so a thread naming one is refused with `400 configuration_unsupported`. See "Publishing Options" above

### A thread is all or nothing

If any part fails to publish, every part that had already published is deleted from the platform and the whole thread's quota is refunded. That holds on every path: immediate, scheduled, promoting a draft, and each account of a multi-account publish. On LinkedIn and Facebook the same rule covers the standalone posts made so far. **Instagram is the exception**: Chirpie cannot delete a published Instagram post, so a failed Instagram thread leaves every part it had already published live and names them, which is what `rollback_supported: false` below means.

The error carries `thread_rollback` saying what was removed, and the code tells you whether a retry is safe:

- `502 upstream_error`: `still_live` is empty. Nothing of the thread is on the platform, so repeating the same call is safe.
- `502 thread_rollback_incomplete`: the posts in `still_live` are really still up, with their ids and public URLs. Repeating the call would publish them twice, so remove them first, or send only the remaining parts.

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

`rollback_supported: false` means Chirpie cannot delete a published post on that platform, so nothing could be removed. That is always the case on Instagram. A **scheduled** thread that fails is retried three times, unless the rollback left posts live: then it fails at once, because a retry would publish them twice.

## Drafts

`draft: true` on `POST /api/v1/posts` or `POST /api/v1/threads` saves the content and sends nothing. The post is stored with `status: "draft"`, never publishes on its own, never enters the scheduler queue, and costs neither the monthly post quota nor the scheduled-post quota until it is promoted.

A draft is held to far less than a post: the text may be empty or absent, media may be missing even where the platform requires it, a draft thread may be 1 to 25 parts where a real thread needs 2, and a `schedule_at` is only the time the customer has in mind, so it need not be in the future. Still refused as always: an account that is not yours or not active (404), a media id you do not hold (`404 media_not_found`), an unknown top-level field, text over 25,000 characters, more than 25 ids in `account_ids`, a duplicate id, and two media spellings at once (400).

```typescript
const draft = await chirpie.createPost({
  account_id: "YOUR_ACCOUNT_ID",
  text: "Half an idea. Finish it later.",
  draft: true,
});
for (const w of draft.warnings) console.log(w.account_id, w.code, w.message);

// Ask for them by name: a listing with no status leaves drafts out.
const drafts = await chirpie.listPosts({ status: "draft" });
```

### `warnings`

Every draft answer carries `warnings`, always present and empty when nothing would go wrong. Each entry is `{ account_id, platform, code, message }` and says what would happen to that account if the draft were sent as it stands. Anything that would really be refused carries the **same code and the same sentence** the API would answer the send with (`bad_request`, `unsupported_media`, `x_link_posts_require_paid_plan`). Three codes describe a change rather than a refusal:

| Code | Meaning |
|------|---------|
| `thread_not_native` | The platform has no reply chain, so each part publishes as its own standalone post |
| `x_link_post_billed` | The post contains a link, so X bills it on top of the monthly quota |
| `schedule_at_in_past` | The remembered time has already passed |

Warnings never stop the save. An edit that leaves the post a draft answers with `warnings` too, rather than an error.

### Promoting a draft

`PATCH /api/v1/posts/:id` promotes, and only when asked:

- `schedule_at` queues it as a real scheduled post at that time.
- `publish: true` publishes it now. Never valid together with `schedule_at` (400).
- `draft: true` alongside `schedule_at` keeps it a draft and only changes the time it remembers.
- `draft: false` is refused with a 400 saying to send `schedule_at` or `publish: true` instead.
- On a post that is not a draft, `publish` and `draft` are both 400.

Promotion runs every rule a create runs (character limits, media rules, X link billing, the 5 minute schedule spacing, the two-post thread minimum) and spends the quota the draft never spent. Anything refused leaves the draft exactly as it was. The answer is the **new** post plus `promoted_from_draft_id`, or `promoted_from_draft_ids` for a draft thread, which is promoted whole: promoting any part queues or publishes all of it. The draft itself is gone, so do not reuse its id.

Deleting a draft refunds nothing, because nothing was charged, and it deletes the whole draft: every account it was addressed to and every part of it, returning `deleted_ids`.

## List Posts

```typescript
const posts = await chirpie.listPosts({
  status: "published",  // Optional: draft|scheduled|publishing|published|failed|deleted
  account_id: "uuid",   // Optional
  group_id: "uuid",     // Optional: every post of one multi-account send
  limit: 20,            // Max 100
  offset: 0,
  include_hidden: false, // Optional: posts the user hid are left out by default
});
```

A post the user deleted stays in the listing with `status: "deleted"`: a delete takes the post down from the platform and never removes the record of it.

## Get a Post

```typescript
const post = await chirpie.getPost("post-uuid");
```

## Delete a Post

```typescript
const result = await chirpie.deletePost("post-uuid");
```

On a published post, delete means delete on the platform: Chirpie tells the platform first and reports the post deleted only once the platform confirms it is gone. If the platform refuses, nothing changes and the call throws `502 upstream_error`: retry the same call. On a post that has not gone out, nothing reaches a platform. A queued post is cancelled and its quota returned, named in `cancelled_ids`. A draft is simply marked deleted, named in `deleted_ids`, and nothing is returned, because a draft never counted against any quota: do not tell the user a deleted draft gave them quota back.

**The post is never removed from Chirpie.** It keeps its id and its history with `status: "deleted"`, so `getPost` still returns it.

A published Instagram or TikTok post, or a published Facebook Page story, cannot be deleted, so it throws `501 delete_unsupported` and nothing changes. The post stays `published` in Chirpie. Tell the user to delete it in the platform's own app, and offer to hide it. A Page story disappears on its own 24 hours after it was posted.

Delete reaches the platform and cannot be undone, so confirm with the user first. Hiding is the reversible alternative.

## Hide a Post

```typescript
await chirpie.hidePost("post-uuid");
await chirpie.unhidePost("post-uuid");
```

Hiding takes a post out of the user's Chirpie listings and nothing else. **Nothing reaches the platform**: the post stays exactly as it is, keeps its analytics and its comments, and `unhidePost` puts it back. Nothing is charged and no quota moves.

Hide is the answer when the user wants a post out of their way; delete is the answer when they want it taken down. A hidden post is left out of `listPosts` unless `include_hidden: true`, and is always readable by id with `getPost`.

**Hiding a queued post does not stop it publishing.** Hide only decides what Chirpie shows; the scheduler pays no attention to it. To stop a scheduled post going out, delete it, which cancels it and returns its quota.

A thread, or a multi-account send, is hidden as the one thing it was made as, so `hidden_ids` can name more posts than the id you passed. Unhiding a post that was never hidden succeeds and changes nothing.

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
| LinkedIn (Page) _(Coming Soon)_ | Yes | Yes | No | Your own comments |
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

// Ask the platform now instead of reading the snapshot. `GET
// /api/v1/analytics/posts/:id?refresh=true` on the wire. Floored at one forced
// refresh per post every 30 minutes; past that it is
// `429 analytics_refresh_rate_limited` carrying Retry-After, and the stored
// numbers are still one ordinary call away.
const fresh = await chirpie.getPostAnalytics("post-uuid", { refresh: true });
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
      case 403: // insufficient_scope: the key lacks the scope named in err.message
      case 404: // Account not found or inactive
      case 409: // idempotency_in_progress: retry the identical call to collect the replay
      case 422: // idempotency_key_reused: use a fresh key for a different request
      case 429: // usage_limit_exceeded (quota), rate_limited (burst), account_limit_reached, or analytics_refresh_rate_limited
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
draft:      → draft (stays there until promoted: schedule_at → scheduled, publish → published)
deleted:    → deleted (also removed from platform if published, except Instagram and Facebook Page stories)
```

Failed posts: check `error_message` for the platform's refusal and `retry_count` for how many attempts were made.

A post with `status: "deleted"` and `error_message` of `"account_disconnected"`, `"account_deactivated"` or `"connection_lost"` was cancelled by Chirpie, not by you, and its quota was returned. `"connection_lost"` means the platform stopped accepting that account's credential: check `inactive_reason` on the account, stop posting to it, and tell the user to reconnect. See https://chirpie.ai/docs/connection-dies.

## Rate Limit Headers

An authenticated `/api/v1/*` response carries `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (unix seconds). Treat them as advisory: the burst limiter fails open, so a response can arrive with none of the three. When they are absent, carry on and do not invent a number in their place. The ceiling is set by the plan (Free 120/min, Agent and Starter 600/min, Pro and Scale 1,200/min), so read it off `X-RateLimit-Limit` rather than assuming a number. A `429` from the burst limiter also carries `Retry-After` (seconds): sleep that long and retry once. A `429` with no `Retry-After` is a quota (`usage_limit_exceeded`) or account-limit (`account_limit_reached`) refusal, so do not retry it.

## Scopes

An API key may be created with `scopes`, in which case it can only reach the routes those scopes cover; a key created without them can do exactly what the key that made it can do, which is full access for a key that has it. Posting needs `posts:write`, reading posts `posts:read`, uploading media `media:write`, comments `comments:read` / `comments:write`, analytics `analytics:read`, accounts `accounts:read` / `accounts:write`, and all three `/api/v1/keys` methods `keys:write` (there is no `keys:read`). A call outside the key's scopes is `403 insufficient_scope`, naming the one that is missing. A key can never grant a scope it does not itself hold.

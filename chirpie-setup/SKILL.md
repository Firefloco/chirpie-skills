---
name: chirpie-setup
description: Set up Chirpie in your project. Install the SDK, configure API keys, connect X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram accounts, send your first post.
---

# Chirpie Setup

## Prerequisites

- A Chirpie account at https://chirpie.ai/auth/signup
- An API key (created during onboarding or at https://chirpie.ai/dashboard/keys)
- At least one connected social account (via https://chirpie.ai/dashboard/accounts)

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

### Scopes: give the key only what it needs

A key created without scopes can do whatever created it: everything, from the dashboard or
from a full-access key, which is the right default for your own project. When you are handing
a key to an agent or a third-party service, narrow it instead. A narrowed key that mints
another without naming scopes gets a copy of itself, never a wider key.

In the dashboard, **Create Key** offers **Full access** or a checkbox list of permissions.
From the CLI or the SDK:

```bash
chirpie keys create -n "Publishing bot" --scope posts:write --scope media:write
```

```typescript
await chirpie.createKey({ name: "Publishing bot", scopes: ["posts:write", "media:write"] });
```

Vocabulary: `posts:read`, `posts:write`, `accounts:read`, `accounts:write`, `analytics:read`,
`comments:read`, `comments:write`, `media:write`, `keys:write`. A call outside the key's
scopes is refused with `403 insufficient_scope` naming the missing one, and a key can never
grant a scope it does not itself hold. All three key methods need `keys:write`: there is no
`keys:read`, because the key list is the inventory of the account's credentials.

## Step 3: Connect Accounts

**X/Twitter**: Connect via OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectXAccount();
// Open URL in browser to authorize
```

Optionally connect X through **your own** X developer app, so posts use your X
API credits and X link posts are not surcharged. Register the app at
developer.x.com with `https://chirpie.ai/api/auth/x/callback` as its callback URI
and the scopes `tweet.read tweet.write users.read offline.access`, then:
```typescript
await chirpie.setXKeys({
  client_id: process.env.X_CLIENT_ID!,
  client_secret: process.env.X_CLIENT_SECRET!,
});
// Then reconnect the X account to move it onto your app.
```
Full walkthrough: https://chirpie.ai/docs/x-byo-keys

**Bluesky**: Connect with an app password (generate at https://bsky.app/settings/app-passwords). The `identifier` takes the first part of the handle on its own (`yourhandle`), a full handle, a handle on your own domain, or the email on the account:
```typescript
await chirpie.connectBlueskyAccount({
  platform: "bluesky",
  identifier: "yourhandle.bsky.social",
  app_password: "xxxx-xxxx-xxxx-xxxx",
});
```

Or via CLI:
```bash
chirpie accounts connect-bluesky --handle yourhandle --app-password xxxx-xxxx-xxxx-xxxx
# --handle is required, --app-password is optional (prompts securely if omitted)
# --handle also accepts yourhandle.bsky.social, a custom-domain handle, or the account email
```

**LinkedIn**: Connect via OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectLinkedInAccount();
// Open URL in browser to authorize via LinkedIn OAuth
```

Or via CLI:
```bash
chirpie accounts connect-linkedin
# Opens browser for LinkedIn OAuth authorization
```

LinkedIn has two kinds of account, connected separately. The call above
connects your own profile. The LinkedIn Pages you administer are a second
connection, coming soon:

```typescript
const { authorization_url } = await chirpie.connectLinkedInPagesAccount();
```

```bash
chirpie accounts connect-linkedin --pages
```

Each Page is an account with its own `id`, and `account_type` says which kind
it is (`"member"` for your profile, `"organization"` for a Page). Post to
either by passing that account's `id`.

**Threads** (coming soon): Connect via Meta OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectThreadsAccount();
// Open URL in browser to authorize via Meta OAuth
```

Or via CLI:
```bash
chirpie accounts connect-threads
# Opens browser for Meta OAuth authorization
```

**Mastodon**: Connect via OAuth from the dashboard or API. The `instance_url` takes a bare host (`mastodon.social`), a full URL, or an `@you@fosstodon.org` address:
```typescript
const { authorization_url } = await chirpie.connectMastodonAccount({
  platform: "mastodon",
  instance_url: "mastodon.social",
});
// Open URL in browser to authorize via Mastodon OAuth
```

Or via CLI:
```bash
chirpie accounts connect-mastodon --instance mastodon.social
# Opens browser for Mastodon OAuth authorization
```

**Instagram** (coming soon): Connect via Instagram Login from the dashboard or API. Requires an Instagram professional account (Business or Creator); personal accounts cannot be connected. Instagram posts always need at least one image.
```typescript
const { authorization_url } = await chirpie.connectInstagramAccount();
// Open URL in browser to authorize via Instagram Login
```

Or via CLI:
```bash
chirpie accounts connect-instagram
# Opens browser for Instagram Login OAuth authorization
```

**Facebook** (coming soon): Connect Facebook Pages via Facebook Login from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectFacebookAccount();
// Open URL in browser to authorize via Facebook Login and choose which Pages to grant
```

Or via CLI:
```bash
chirpie accounts connect-facebook
# Opens browser for Facebook Login OAuth authorization
```

One authorization can grant many Pages ("all current and future Pages" grants every Page you
manage). Each granted Page becomes its own Chirpie account. If you grant more Pages than your
plan's account limit allows, Chirpie switches on as many as it can and stores the rest with
`is_active: false` and `inactive_reason: "plan_limit"`, so nothing is dropped. Choose which
Pages publish:
```typescript
await chirpie.deactivateAccount(currentlyActiveId);  // frees a plan slot
await chirpie.activateAccount(parkedPageId);
```

To end a connection altogether, `await chirpie.disconnectAccount(id)` (CLI:
`chirpie accounts disconnect <id>`, MCP: `chirpie_disconnect_account`). It frees a plan slot
like deactivating and cancels the account's scheduled posts the same way, but the stored
credential is removed, so connecting the account again means authorizing it on the platform
again. Posts it already published are kept. Confirm with the user first: it cannot be undone.

Reconnecting Facebook re-runs the import, so Pages added later are picked up without
disconnecting anything.

**Telegram**: Connect with a bot token (create via [@BotFather](https://t.me/BotFather)). The `chat_id` takes a bare channel name (`channelname`), `@channelname`, a `https://t.me/channelname` link, or the numeric chat ID:
```typescript
await chirpie.connectTelegramAccount({
  platform: "telegram",
  bot_token: "123456789:ABCdefGHIjklMNOpqrSTUvwxYZ",
  chat_id: "-1001234567890",
});
```

Or via CLI:
```bash
chirpie accounts connect-telegram --bot-token TOKEN --chat-id channelname
# --bot-token and --chat-id are optional (prompts securely if omitted)
# --chat-id also accepts @channelname, a t.me link, or a numeric chat ID
```

**Threads, Instagram, Facebook, Pinterest, TikTok, YouTube, Google Business Profile**: Coming soon. Connecting one today returns `501 platform_coming_soon`. Accounts already connected keep posting, scheduling and reporting analytics as normal.

## Step 4: Find Your Account ID

```typescript
const accounts = await chirpie.listAccounts();
console.log(accounts);
// listAccountsWithLimits() returns the same accounts plus accounts_limit
// (your plan's max, or null when the plan sets no limit) and accounts_active
// (how many are currently on).
// Each account has: id, platform, username, display_name, avatar_url, is_active, and inactive_reason when it is switched off. `plan_limit` means activate it once a slot is free; `token_revoked`, `token_expired` and `reauth_required` mean it has to be connected again at https://chirpie.ai/dashboard/accounts; no reason at all means it was switched off deliberately and activating is enough
// Use the `id` field as your account_id for posting.
// Accounts with is_active: false are not publishing. inactive_reason: "plan_limit"
// means the account is connected but over your plan's account limit. Activate it
// with chirpie.activateAccount(id) rather than reconnecting.
```

## Step 5: Send Your First Post

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
2. **Account not active:** If OAuth tokens expired or were revoked, `is_active` will be `false`. Re-authorize from the dashboard. For Bluesky, generate a new app password. For Telegram, verify bot token and chat ID.
3. **Environment variable loading:** In Next.js, server-side env vars don't need `NEXT_PUBLIC_` prefix. Don't expose your API key to the browser.
4. **Platform names are validated.** `POST /api/v1/accounts` rejects an unrecognised `platform` with `400 bad_request` (for example `twitter`; the value is `x`). Omitting `platform` still means `x`.
5. **Account limit applies to new connections only.** Connecting a platform you have no account on returns `429 account_limit_reached` when your plan is full. Reconnecting a platform you already hold (re-authorizing an expired X token, or moving an X account onto your own developer app) needs no free slot and works on every plan, Free included.

## Plan Limits

| Plan | Posts/mo | Scheduled/mo | Accounts | Price |
|------|----------|-------------|----------|-------|
| Free | 50 | 25 | 1 | $0 |
| Agent | 300 | 150 | 1 | $9/mo |
| Starter | 1,000 | 500 | 3 | $19/mo |
| Pro | 5,000 | 2,500 | 10 | $49/mo |
| Scale | 25,000+ | 12,500+ | 25+ | Custom |

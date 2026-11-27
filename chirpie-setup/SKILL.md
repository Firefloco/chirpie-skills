---
name: chirpie-setup
description: Set up Chirpie (chirpie.ai). For an AI app, add the hosted MCP server at https://chirpie.ai/mcp in one step; for code, install the SDK, configure API keys, connect X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram accounts, send your first post.
---

# Chirpie Setup

## First: which route?

- **The user wants their AI app (Claude, Claude Code, Cursor, VS Code, ChatGPT, Codex, Gemini CLI, Windsurf, Manus) to post.** Use the hosted MCP server, `https://chirpie.ai/mcp`. No SDK, and no API key in any app that signs in to Chirpie with OAuth (Manus is the exception: give it a scoped key as a bearer token, see `chirpie-mcp`). In Claude Code that is one command:

  ```bash
  claude mcp add --transport http chirpie https://chirpie.ai/mcp
  ```

  then `/mcp` to sign in. A person with no Chirpie account signs up in that window with just their email and a 6-digit code (no password) and goes straight back to the app. Every other app is in the `chirpie-mcp` skill and at https://chirpie.ai/docs/mcp?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup. Connect social accounts at https://chirpie.ai/dashboard/accounts?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup, and stop here.
- **A shell agent, a script or a scheduled job.** Use the CLI: `npm install -g chirpie`, `chirpie login` (prints a link and a code for the person to approve in any browser, so it works over SSH). See `chirpie-cli`.
- **You are the agent and the person has no key or account yet.** Use device authorization, below: the person taps Approve once and you receive a key.
- **The user is building an app or a service.** Use the SDK or the REST API with an API key: the rest of this skill.

## If you are the agent doing the setup

You can get the person started without handling any credential; they only tap **Approve** on a link you relay. Never ask them for a password, an emailed code or an API key.

1. `POST https://chirpie.ai/api/v1/device/code` with `{"client_name": "<you> on <host>"}`, no auth. If the person has no Chirpie account you may instead `POST https://chirpie.ai/api/v1/signup` with `{"email": "<their address>"}` (ask them for it): they get one approval email.
2. Relay `data.human_action.message` to the person exactly as written. It names the link and the code.
3. Poll `POST https://chirpie.ai/api/v1/device/token` with `{"device_code": "..."}` every `data.interval` seconds. `authorization_pending`: keep waiting. `slow_down`: use the new `interval`. The first `200` carries `data.api_key` (`chirpie_sk_...`), once.
4. Use it as `Authorization: Bearer <key>`. If `GET /api/v1/accounts` lists nothing, `POST /api/v1/accounts` with a `platform` and relay the `human_action` it returns.

Any later answer or error that carries `human_action` (a connect link, a plan limit) is a step only the person can take: relay it the same way. `chirpie login` does steps 1 to 3 in a terminal, over SSH too. In Claude Code, Codex and Gemini CLI, whose hosted-server sign-in is a command only the person can run (`/mcp`, `codex mcp login`, `/mcp auth`), add the hosted server with the key instead (see `chirpie-mcp`). Full guide: https://chirpie.ai/docs/agents?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup

## Prerequisites (SDK and REST API)

- A Chirpie account at https://chirpie.ai/auth/signup?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup (email and a 6-digit code; a password is optional and can be added later in Settings)
- An API key (shown during onboarding, created at https://chirpie.ai/dashboard/keys?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup, or written for you by `chirpie login`, which also works for a brand-new, signed-out user)
- At least one connected social account (via https://chirpie.ai/dashboard/accounts?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup)

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
// A chirpie.ai link: open it in a browser, sign in to Chirpie if asked,
// press Continue and approve on X.
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
Full walkthrough: https://chirpie.ai/docs/x-byo-keys?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup

**Bluesky**: Connect with an app password, never the account password. Create one in Bluesky under Settings, Privacy and Security, App Passwords (https://bsky.app/settings/app-passwords); it looks like `xxxx-xxxx-xxxx-xxxx`, and anything else is refused with `400 bluesky_app_password_required` (the CLI refuses it locally with the same code). The `identifier` takes the first part of the handle on its own (`yourhandle`), a full handle, a handle on your own domain, or the email on the account:
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

**Threads**: Connect via Meta OAuth from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectThreadsAccount();
// Open URL in browser to authorize via Meta OAuth
```

Or via CLI:
```bash
chirpie accounts connect-threads
# Opens browser for Meta OAuth authorization
```

**Mastodon**: Connect via OAuth from the dashboard or API. The `instance_url` takes the server (`mastodon.social`), the user's handle (`@you@mastodon.social`) or their profile URL (`https://mastodon.social/@you`), and keeps only the server. A value naming no server is `400 mastodon_instance_invalid`:
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
# --instance also takes @you@mastodon.social or https://mastodon.social/@you
# Prints the URL to open for Mastodon OAuth authorization
```

**Instagram**: two routes, and the user picks. Both require an Instagram professional account (Business or Creator); personal accounts cannot be connected. Instagram posts always need media: a feed post takes one image or a carousel of up to 10, a story takes exactly one image or video, and a reel takes exactly one video. Text on its own is refused.

- `via: "instagram"` (the default): the user signs in with Instagram. No Facebook Page needed.
- `via: "facebook"`: the user signs in with Facebook, and Chirpie connects the Instagram professional accounts linked to the Facebook Pages they share. One authorization can connect several at once, one per linked Page, and any beyond the plan's account limit arrive inactive with `inactive_reason: "plan_limit"` rather than dropped.

Posting, carousels, stories, reels, threads, scheduling, analytics and comments are identical on both. The only difference is deleting a published post: the Facebook route can, the Instagram route answers `501 delete_unsupported`. Recommend `via: "facebook"` when the user wants deletes or runs several accounts across Pages.

Connecting an account the user already has through the other route **moves** the account they already have rather than adding a second one: it keeps its id and everything it has published, and only deleting changes hands. So switching to `via: "facebook"` adds deleting, and switching back takes it away. Where the two routes report different Instagram accounts the user gets a second account instead, so list the accounts again afterwards rather than assuming which happened.

```typescript
const { authorization_url } = await chirpie.connectInstagramAccount();
// Open URL in browser to authorize

// Or sign in with Facebook instead:
const viaFacebook = await chirpie.connectInstagramAccount({ via: "facebook" });
```

Or via CLI:
```bash
chirpie accounts connect-instagram
chirpie accounts connect-instagram --via facebook
# Opens browser for OAuth authorization
```

**Facebook**: Connect Facebook Pages from the dashboard or API:
```typescript
const { authorization_url } = await chirpie.connectFacebookAccount();
// Open URL in browser to authorize and choose which Pages to grant
```

Or via CLI:
```bash
chirpie accounts connect-facebook
# Opens browser for Facebook OAuth authorization
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

**Pinterest, TikTok, YouTube, Google Business Profile**: Coming soon. Connecting one today returns `501 platform_coming_soon`.

## Step 4: Find Your Account ID

```typescript
const accounts = await chirpie.listAccounts();
console.log(accounts);
// listAccountsWithLimits() returns the same accounts plus accounts_limit
// (your plan's max, or null when the plan sets no limit) and accounts_active
// (how many are currently on).
// Each account has: id, platform, username, display_name, avatar_url, is_active, and inactive_reason when it is switched off. `plan_limit` means activate it once a slot is free; `token_revoked`, `token_expired` and `reauth_required` mean it has to be connected again at <https://chirpie.ai/dashboard/accounts?utm_source=agent_skills&utm_medium=skill&utm_campaign=chirpie-setup>; no reason at all means it was switched off deliberately and activating is enough
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

Instagram is the one platform that refuses a text-only post: add `media_urls` with at least one public image URL when the account is an Instagram one.

## Alternative: CLI Setup

```bash
# Install globally
npm install -g chirpie

# Prints a link and a code; approve in any browser (creates the API key)
chirpie login
# No account yet: one approval email, then the same wait
chirpie signup --email you@example.com

# Post (auto-selects account if you only have one)
chirpie post "Hello from the CLI!"
```

## Alternative: MCP Setup

For any AI app, the hosted server at `https://chirpie.ai/mcp` needs no code at all. See the `chirpie-mcp` skill.

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

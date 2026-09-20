---
name: chirpie-cli
description: Use the Chirpie CLI to post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram from the terminal, one account or several at once. Covers installation, browser-based login, drafts, and all commands.
---

# Chirpie CLI

## Installation

```bash
npm install -g chirpie
```

Or use with npx:

```bash
npx chirpie post "Hello!"
```

## Authentication

### Browser Login (Recommended)

```bash
chirpie login
```

Opens your browser, signs you in, and saves an API key to `~/.chirpie/config.json`.

### Manual Key

```bash
chirpie auth --key chirpie_sk_YOUR_KEY
```

### Environment Variable

```bash
export CHIRPIE_API_KEY=chirpie_sk_YOUR_KEY
```

`CHIRPIE_API_KEY` env var takes precedence over the config file.

### Check Status

```bash
chirpie whoami
```

### Logout

```bash
chirpie logout
```

## Commands

### chirpie post

```bash
chirpie post "Your tweet text"
chirpie post "Scheduled!" -s "2026-04-01T14:00:00Z"
chirpie post "Specific account" -a ACCOUNT_UUID
chirpie post "With a local file" -m ./shot.png --alt "The new dashboard"
chirpie post "Half an idea" --draft
chirpie post "JSON output" --json
chirpie post "To two accounts at once" -a ACCOUNT_A -a ACCOUNT_B
chirpie post "With its own text for one" -a ACCOUNT_A -a ACCOUNT_B \
  --config '{"ACCOUNT_B":{"text":"Text for just this account"}}'
```

| Flag | Description |
|------|-------------|
| `-a, --account <id>` | Account ID (auto-selects if only one). Repeat it to publish to several accounts in one call |
| `--config <json-or-file>` | Per-account overrides for a multi-account post, as JSON or a path to a JSON file. Keyed by account ID, each value taking `text` and media. A field left out inherits the call's own; media replaces rather than merges |
| `-m, --media <files...>` | Image or video files on this machine, or public URLs. Files are uploaded first |
| `--alt <text...>` | Describe each item for screen readers, in the same order as `--media` |
| `-s, --schedule <datetime>` | ISO 8601 datetime with a timezone (`...Z` or `+02:00`); normalized to UTC |
| `--draft` | Save without publishing. Nothing is sent and nothing counts against the quota. Anything that would go wrong is printed, one warning per line |
| `--json` | Machine-readable JSON output |

### chirpie thread

```bash
chirpie thread "First post" "Second post" "Third post"
chirpie thread "Post 1" "Post 2" -s "2026-04-01T14:00:00Z"
chirpie thread "First post" "Second post" -a ACCOUNT_A -a ACCOUNT_B \
  --config '{"ACCOUNT_B":{"posts":[{"text":"a"},{"text":"b"},{"text":"c"}]}}'
```

Min 2 posts, max 25, or 1 to 25 with `--draft`. Same flags as `chirpie post`. Repeat `-a` to publish the thread to several accounts at once; a `--config` override's `posts` replaces the whole thread for that account.

A multi-account send reports per account: some can publish while others fail, and an account that fails gives its quota back. A problem the platform rules catch up front (character limit, media rules, the X link-post rule) refuses the whole call and publishes nothing. Full detail: https://chirpie.ai/docs/multi-account

### chirpie posts

```bash
chirpie posts                           # List recent posts
chirpie posts --status published        # Filter by status
chirpie posts --limit 50               # More results
chirpie posts --group GROUP_UUID       # Every post of one multi-account send
chirpie posts get POST_UUID            # Get single post
chirpie posts update POST_UUID --text "Fixed"   # Edit a queued post, keeping its time
chirpie posts update POST_UUID --schedule-at 2027-04-02T09:00:00Z  # Move it
chirpie posts delete POST_UUID         # Take it down from the platform
chirpie posts hide POST_UUID           # Hide it from Chirpie only, reversibly
chirpie posts unhide POST_UUID         # Show it again
chirpie posts --include-hidden         # Include the ones that are hidden
chirpie posts --json                   # JSON output
```

On a published post, delete takes it down from the platform and succeeds only once the platform confirms it is gone. On one that has not gone out, nothing reaches a platform: a queued post is cancelled and its quota returned, while a draft is simply marked deleted, since a draft never counted against any quota. Chirpie keeps the post either way, marked deleted, so `chirpie posts --status deleted` still lists it. Instagram and TikTok publish no delete API, so a published post there is refused with `delete_unsupported`: delete it in the platform's own app.

Hide reaches no platform at all. It only decides whether Chirpie shows the post, and `unhide` is the exact undo. A thread, or a multi-account send, is hidden as the one thing it was made as.

A thread is atomic: if any part fails, the parts already published are deleted and the quota refunded. If a part could not be removed the CLI prints it under "Still live on the platform, delete these yourself", and the code is `thread_rollback_incomplete` rather than `upstream_error`, so do not retry blindly.

### Drafts

```bash
chirpie post "Half an idea" --draft     # Save it, send nothing
chirpie thread "Opening line" --draft   # A draft thread may be one post
chirpie posts --status draft            # See what is saved
chirpie posts update POST_UUID --schedule-at 2027-04-02T09:00:00Z --keep-draft  # Change only the remembered time
chirpie posts update POST_UUID --schedule-at 2027-04-02T09:00:00Z  # Promote: queue it
chirpie posts publish POST_UUID         # Promote: send it now
chirpie posts publish POST_UUID -t "The final wording"
```

A draft reaches no platform and costs no quota until it is promoted. Saving one
prints, one per line, anything that would go wrong if it were sent as it stands.
Promotion runs every rule a create runs and takes the quota, so a draft that
would be refused stays a draft, unchanged. A draft thread is promoted whole.
`--keep-draft` is valid on its own as well.

### chirpie accounts

```bash
chirpie accounts                       # List connected accounts (inactive ones included)
chirpie accounts activate <id>         # Switch an account on so it can publish
chirpie accounts deactivate <id>       # Switch it off (frees a plan slot, CANCELS its scheduled posts)
chirpie accounts disconnect <id>       # End the connection (asks first; -y to skip, CANCELS its scheduled posts)
chirpie accounts connect-x             # Start X OAuth flow (prints URL)
chirpie accounts connect-bluesky --handle yourhandle --app-password xxxx-xxxx-xxxx-xxxx  # --handle also takes a full handle, a custom domain, or the account email
chirpie accounts connect-linkedin     # Start LinkedIn OAuth flow (prints URL to open in browser)
chirpie accounts connect-linkedin --pages  # Connect the LinkedIn Pages you administer instead (COMING SOON)
chirpie accounts connect-threads      # Start Threads Meta OAuth flow (COMING SOON)
chirpie accounts connect-mastodon --instance mastodon.social  # Start Mastodon OAuth flow (a full URL or @you@server also works)
chirpie accounts connect-instagram     # Start Instagram Login OAuth flow (COMING SOON)
chirpie accounts connect-facebook      # Start Facebook Login OAuth flow (COMING SOON)
chirpie accounts connect-telegram --bot-token TOKEN --chat-id channelname  # Connect Telegram bot (@channelname, a t.me link, or a numeric ID also work)
# Threads, Instagram, Facebook, Pinterest, TikTok, YouTube, and Google Business
# Profile are coming soon. Their connect-* commands answer with a "coming soon"
# message. Accounts already connected keep posting and scheduling as normal.

# Optional: connect X through your OWN X developer app, so posts use your X API
# credits and X link posts are not surcharged.
# Setup guide: https://chirpie.ai/docs/x-byo-keys
chirpie accounts x-keys set --client-id ID   # Prompts for the secret (no echo)
chirpie accounts x-keys status               # Shows config status, never the secret
chirpie accounts x-keys remove
# Then run `chirpie accounts connect-x` to move an account onto your app.
```

One Facebook authorization can grant several Pages at once. Every granted Page is imported
as its own account; any beyond the plan's account limit are stored inactive (shown as
`over plan limit`) rather than dropped. Use `chirpie accounts activate <id>` to choose which
Pages publish, and `chirpie accounts deactivate <id>` to free a slot for a different one.
Re-running `chirpie accounts connect-facebook` re-imports, picking up newly added Pages.

Deactivating cancels the account's scheduled posts (a switched-off account cannot publish),
returning them to the monthly quota. The `scheduled` column in `chirpie accounts` shows how
many would go; activating the account again does not bring them back. Confirm with the user
before deactivating an account whose `scheduled` count is above zero.

`chirpie accounts disconnect <id>` cancels that queue the same way, and goes further: it ends
the connection, so the stored credential is removed and connecting the account again means
authorizing it on the platform again. Posts it already published are kept. It cannot be
undone, so it asks first; pass `-y` in a non-interactive shell or it stops without
disconnecting anything. Prefer `deactivate` when the account should come back later.

### chirpie keys

```bash
chirpie keys                          # List API keys
chirpie keys create                   # Create new key
chirpie keys create -n "Bot Key"      # Create with name
chirpie keys revoke KEY_UUID          # Revoke a key
```

### chirpie analytics

```bash
chirpie analytics POST_UUID           # Get post metrics
```

## Output Formats

By default, output is human-readable (tables and messages). Add `--json` for machine-readable output:

```bash
chirpie posts --json | jq '.[].text'
```

On a subcommand the flag works in either position, before or after the arguments:

```bash
chirpie accounts deactivate ACCOUNT_UUID --json    # includes canceled_posts
chirpie accounts --json deactivate ACCOUNT_UUID    # identical
chirpie accounts disconnect ACCOUNT_UUID -y --json  # includes disconnected and canceled_posts
```

## Config File

Location: `~/.chirpie/config.json` (0600 permissions)

```json
{
  "api_key": "chirpie_sk_...",
  "base_url": "https://chirpie.ai"
}
```

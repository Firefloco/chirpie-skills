---
name: chirpie-cli
description: Use the Chirpie CLI to post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, Facebook, and Telegram from the terminal, one account or several at once. Covers installation, browser-based login, and all commands.
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
| `--json` | Machine-readable JSON output |

### chirpie thread

```bash
chirpie thread "First post" "Second post" "Third post"
chirpie thread "Post 1" "Post 2" -s "2026-04-01T14:00:00Z"
chirpie thread "First post" "Second post" -a ACCOUNT_A -a ACCOUNT_B \
  --config '{"ACCOUNT_B":{"posts":[{"text":"a"},{"text":"b"},{"text":"c"}]}}'
```

Min 2 posts, max 25. Same flags as `chirpie post`. Repeat `-a` to publish the thread to several accounts at once; a `--config` override's `posts` replaces the whole thread for that account.

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
chirpie posts delete POST_UUID         # Delete a post
chirpie posts --json                   # JSON output
```

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

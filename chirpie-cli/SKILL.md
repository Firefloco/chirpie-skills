---
name: chirpie-cli
description: Use the Chirpie CLI to post to X/Twitter, Bluesky, LinkedIn, Threads, Mastodon, Instagram, and Facebook from the terminal. Covers installation, browser-based login, and all commands.
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
chirpie post "JSON output" --json
```

| Flag | Description |
|------|-------------|
| `-a, --account <id>` | Account ID (auto-selects if only one) |
| `-s, --schedule <datetime>` | ISO 8601 datetime (UTC) |
| `--json` | Machine-readable JSON output |

### chirpie thread

```bash
chirpie thread "First post" "Second post" "Third post"
chirpie thread "Post 1" "Post 2" -s "2026-04-01T14:00:00Z"
```

Min 2 posts, max 25. Same flags as `chirpie post`.

### chirpie posts

```bash
chirpie posts                           # List recent posts
chirpie posts --status published        # Filter by status
chirpie posts --limit 50               # More results
chirpie posts get POST_UUID            # Get single post
chirpie posts delete POST_UUID         # Delete a post
chirpie posts --json                   # JSON output
```

### chirpie accounts

```bash
chirpie accounts                       # List connected accounts (X, Bluesky, LinkedIn, Threads, Mastodon, Instagram, and Facebook)
chirpie accounts connect              # Start X OAuth flow (prints URL)
chirpie accounts connect-bluesky --handle yourhandle.bsky.social --app-password xxxx-xxxx-xxxx-xxxx
chirpie accounts connect-linkedin     # Start LinkedIn OAuth flow (prints URL to open in browser)
chirpie accounts connect-threads      # Start Threads Meta OAuth flow (prints URL to open in browser)
chirpie accounts connect-mastodon --instance https://mastodon.social  # Start Mastodon OAuth flow
chirpie accounts connect-instagram     # Start Instagram Login OAuth flow (prints URL to open in browser)
chirpie accounts connect-facebook      # Start Facebook Login OAuth flow (prints URL to open in browser)
```

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

## Config File

Location: `~/.chirpie/config.json` (0600 permissions)

```json
{
  "api_key": "chirpie_sk_...",
  "base_url": "https://chirpie.ai"
}
```

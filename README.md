# command-tester

Bubbletea TUI for verifying [tripbot](https://github.com/adanalife/tripbot)'s chat command surface end-to-end. Sends commands through Twitch IRC as a configured test user, captures the bot's replies, and prompts the human for Y/N on commands with stream-visible (overlay) effects.

Designed for **one command per keypress** — no batch mode, so it doesn't elbow into Twitch chat rate limits during a test session.

## Run

You need a Twitch developer application registered as a **public** client (no client secret) so the device-code OAuth flow works. Visit [dev.twitch.tv/console/apps](https://dev.twitch.tv/console/apps), register one with category "Chat Bot" and any redirect URL (the device flow doesn't use it). Copy the client ID.

```sh
go build -o command-tester .
TWITCH_CLIENT_ID=<your client id> ./command-tester
```

First run prints a `https://twitch.tv/activate` URL and a user code — open the URL logged into the Twitch account you want to send chat as, paste the code, authorize. Token is cached at `.token.json` (gitignored, mode `0600`); subsequent runs skip the auth flow until the cached token is invalidated.

OAuth scopes requested: `chat:read chat:edit user:read:follows user:read:subscriptions`. The follow / subscriber checks power the role badges in the TUI header so you can tell at a glance whether a follower-gated command is expected to pass.

## Flags

| Flag | Default | Purpose |
|---|---|---|
| `-client-id` | `$TWITCH_CLIENT_ID` | Twitch app client ID |
| `-channel` | `adanalife_staging` | channel to send commands in |
| `-bot` | `tripbot4000` | bot username whose replies to capture |
| `-log` | `logs/command-tester-<date>.log` | JSON-lines session log |
| `-timeout` | `5` | seconds to wait per command |
| `-token-cache` | `.token.json` | OAuth token cache location |

## Keys

**List view**

- `↑` / `↓` move cursor · `g` / `G` first / last
- `space` toggle whether the row runs · `a` / `n` include all / none
- `enter` or `r` run the focused command
- `tab` jump to summary
- `q` quit

**Review screen** (shown after each run)

- `enter` accept the auto-resolved status and advance
- `y` / `n` override as pass / fail · `s` skip
- `c` enter comment mode — type freely, `esc` to exit; notes are saved with the result
- `r` re-run this command
- `esc` back to list without saving

## Log format

JSON-lines at the configured log path. Each entry:

```json
{
  "ts": "2026-05-03T17:55:08Z",
  "session_id": "ab12cd34ef567890",
  "trigger": "!leaderboard",
  "status": "manual-fail",
  "bot_reply": "Right now only followers of the channel can run unlimited commands :)",
  "notes": "failed because test account isn't following",
  "latency_ms": 142,
  "login": "mathgaming",
  "channel": "adanalife_staging",
  "bot": "tripbot4000",
  "command_tester_version": "0ac5590-dirty",
  "follower": false,
  "subscriber": false
}
```

Resume-aware: re-launching reads the existing log on startup and pre-marks rows that already have results, so you can pick up where you left off.

## Catalog

The list of commands and their metadata lives in `catalog.go`, populated by hand from tripbot's `pkg/chatbot/handlers.go` (~28 commands, ~70 aliases). Each entry carries category, expected-reply hint, onscreen-effect tag, and sample params.

**Catalog drift** is a known cost of this living in its own repo. When tripbot grows a new chat command, the catalog here won't auto-track it; you'll notice the gap on the next session. There's open work to either auto-generate the catalog or add a CI check comparing triggers against tripbot's handlers.

## History

Extracted from `adanalife/tripbot`'s `streamtest/` subdir on 2026-05-03 via `git subtree split`, preserving the original four-commit history. The rename commit on top retitles the project from `streamtest` to `command-tester`.

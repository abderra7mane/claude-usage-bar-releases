# Claude Usage Bar

A macOS menu bar app that shows your Claude plan usage for the 5-hour session and the weekly window.

This repo hosts Claude Usage Bar's releases: the installers and the appcasts the app checks for updates.

- Blue: below 75%
- Orange: 75–99%
- Red: limit reached

Console and Studio use their own colors for the same levels (see [Themes](#themes)).

The top bar in the icon is the session, the bottom bar is the week. Click it for details and reset times.

## Notifications

The app notifies you when the session or weekly usage passes 75%, 80%, 85%, 90%, 95% and 100%. If usage jumps past several of them between two checks, you get one notification for the highest. Turn them off in Settings.

## Sessions

Claude Code's prompt cache expires 1 hour after a session's last request; the next message then re-writes the whole context at a higher price. The panel's **Sessions** tab shows one bar per running session with a live cache, full with an hour left and empty when it expires, next to the session's cost. The app notifies you 5 minutes before a session's cache expires, once per expiry, so you can send it a message to keep it. Turn the alerts off in Settings.

The cost is estimated at API prices from the token usage in the session's transcripts, subagents included. It can read lower than Claude Code's own total: subagent transcripts often keep only the usage reported when a response started, and a few requests (title generation, away summaries) aren't recorded in transcripts.

## Multiple subscriptions

Each account is a Claude Code config folder (`CLAUDE_CONFIG_DIR`), e.g. `~/.claude` and `~/.claude-work`. Manage them in **Settings…**: add folders, pick from detected `~/.claude*` folders, rename them, and choose which ones get a menu bar item. Accounts that are logged out or have an expired token are dimmed and offer a "Copy login command" button.

## Themes

The panel and Settings come in three themes, picked in **Settings › Appearance**:

- **Native** (default): the standard macOS look.
- **Console**: dense and monospace, with tick meters, a sessions table and keyboard shortcuts (`1`/`2` switch tabs, `⌘R` refreshes).
- **Studio**: large numbers in cards, gradient bars and a countdown ring per session's cache.

Each theme comes in light and dark; **System / Light / Dark** next to the theme picker follows the system or forces one.

## Install

Download the DMG for your Mac from the [latest release](https://github.com/abderra7mane/claude-usage-bar-releases/releases/latest): `ClaudeUsageBar-arm64.dmg` for Apple Silicon, `ClaudeUsageBar-x86_64.dmg` for Intel or `ClaudeUsageBar-universal.dmg` for both. Open it and drag the app to Applications. The app isn't notarized, so clear the quarantine flag before the first launch:

```sh
xattr -dr com.apple.quarantine /Applications/ClaudeUsageBar.app
```

The app checks for updates daily and installs them from the panel's **Check for Updates…**. Automatic checks can be turned off in Settings.

## Requirements

- macOS 14+
- Claude Code logged in with a Pro/Max subscription (`/login`)

## How it works

- Reads each folder's OAuth token from the keychain via `/usr/bin/security`: `Claude Code-credentials` for `~/.claude`, `Claude Code-credentials-<first 8 hex chars of sha256(folder path)>` for other folders. Falls back to `<folder>/.credentials.json`.
- Polls `GET https://api.anthropic.com/api/oauth/usage` every 3 minutes by default (configurable in Settings), the same undocumented endpoint Claude Code's `/usage` uses. It may change without notice.
- Finds sessions from the transcripts in each folder's `projects/`: the timestamp of the last assistant message is the last request, read from the end of transcripts changed in the past hour, every 30 seconds. Sessions in paid usage overage use a 5-minute cache, which the timers don't account for.
- The token is never refreshed by this app, because refreshing rotates it and would log Claude Code out. When it expires, the app shows the last values and asks you to run `claude`.

## Updating from 0.4.1 or earlier

Versions up to 0.4.1 look for updates in a repo that is no longer public, so they don't find new versions. Download the latest DMG from this repo once and replace the app in Applications; your settings stay. Updates install from the panel after that.

## License

Free to use; all rights reserved. See [LICENSE](LICENSE).

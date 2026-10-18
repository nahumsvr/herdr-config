# Agent status icons

`bin/herdr-status-icons` draws an animated glyph before each agent title in the
sidebar. herdr 0.9.4 only offers static symbols (`status_indicators = "symbols"`),
so the script publishes a custom `$line` token per pane through
`herdr pane report-metadata`, and `config.toml` renders it.

| State | Glyph | Notes |
| --- | --- | --- |
| working | `⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏` | cli-spinners `dots`, 80 ms per frame, frame taken from the clock |
| blocked | `?` | label `input` |
| done | `✓` | finished and not seen yet |
| idle | `⏾` | |
| unknown | `◌` | |

The state text (`working`, `input`, `done`, `idle`) is set with `--state-label`
and shown before the workspace name.

## How it fits together

- `config.toml`: `[ui.sidebar.agents]` first row is `{token = "$line"}`. Glyph and
  title live in one token because herdr draws a ` · ` separator between tokens.
- The script reads `herdr api snapshot` every ~0.25 s and republishes a pane only
  when its line changes, or every 1.6 s so the 15 s TTL never expires. If the
  script dies, icons disappear on their own.
- `done` comes from the server's "seen" state, so it clears when the agent is
  focused through herdr.

## Run it at login (macOS)

Save as `~/Library/LaunchAgents/local.herdr.status-icons.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>local.herdr.status-icons</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/zsh</string>
    <string>/Users/YOU/.config/herdr/bin/herdr-status-icons</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/Users/YOU/.local/bin:/opt/homebrew/bin:/usr/bin:/bin</string>
  </dict>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
  <key>ProcessType</key><string>Background</string>
  <key>StandardErrorPath</key><string>/tmp/herdr-status-icons.log</string>
</dict>
</plist>
```

Replace `YOU`, then:

```sh
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/local.herdr.status-icons.plist
launchctl kickstart -k gui/$(id -u)/local.herdr.status-icons   # restart after edits
launchctl bootout gui/$(id -u)/local.herdr.status-icons        # stop
```

Requires `jq` and `herdr` on the `PATH`.

## Braille alignment in Ghostty

With a large `adjust-cell-height` the braille spinner can sit above the title
baseline. Ghostty draws braille itself, so try mapping it to a real font in the
Ghostty config (applies to new terminals only):

```
font-codepoint-map = U+2800-U+28FF=Apple Braille
```

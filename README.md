# clangine-de-poitrine

Music that plays while Claude Code is cooking. Stops when it needs you.

## How it works

Claude Code hooks trigger shell scripts:
- `UserPromptSubmit` → `clangine play` (you hit enter, music starts)
- `PostToolUse` → `clangine play` (resumes after permission was granted)
- `Stop` → `clangine pause` (Claude finishes, music stops)
- `Notification` → `clangine pause` (Claude needs your attention)
- `PermissionRequest` → `clangine pause` (Claude needs permission)

Controls the Spotify desktop app via AppleScript (macOS).

## Quick start

```bash
# 1. Install
git clone <this-repo> ~/.clangine-de-poitrine
cd ~/.clangine-de-poitrine
chmod +x clangine

# 2. Install hooks into Claude Code
./clangine install

# 3. That's it — next time you use Claude Code, music plays while it thinks
```

## Commands

```bash
clangine play      # start playing
clangine pause     # pause playback
clangine stop      # stop and reset
clangine skip      # next track
clangine status    # show current state
clangine install   # add hooks to ~/.claude/settings.json
clangine uninstall # remove hooks from settings
```

## Configuration

Create `config.sh` in the clangine directory:

```bash
CLANGINE_PLAYLIST="spotify:playlist:37i9dQZF1DX..."  # playlist or album URI
CLANGINE_SHUFFLE="true"                               # randomize track order
```

To get a playlist URI: Spotify → right-click playlist → Share → Copy Spotify URI.

If no playlist is set, it resumes whatever was last playing. If Spotify isn't open, clangine launches it automatically.

By default it plays [Angine de Poitrine — Vol. II](https://anginedepoitrine.bandcamp.com/).

---

*Named after [Angine de Poitrine](https://anginedepoitrine.bandcamp.com/) — microtonal math rock from Saguenay, QC. The perfect coding soundtrack.*

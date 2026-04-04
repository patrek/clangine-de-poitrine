# clangine-de-poitrine

Music that plays while Claude Code is cooking. Stops when it needs you.

## How it works

Claude Code hooks trigger shell scripts:
- `UserPromptSubmit` → `clangine play` (you hit enter, music starts)
- `Stop` → `clangine pause` (Claude finishes, music stops)
- `Notification` → `clangine pause` (Claude needs your attention)

## Quick start

```bash
# 1. Install
git clone <this-repo> ~/.clangine-de-poitrine
cd ~/.clangine-de-poitrine
chmod +x clangine

# 2. Add your tracks
# Drop .mp3/.wav/.ogg files into the tracks/ folder
# A placeholder tone is included for testing

# 3. Install hooks into Claude Code
./clangine install

# 4. That's it — next time you use Claude Code, music plays while it thinks
```

## Commands

```bash
clangine play      # start playing (random track from tracks/)
clangine pause     # pause playback
clangine stop      # stop and reset
clangine skip      # next track
clangine status    # show current state
clangine install   # add hooks to ~/.claude/settings.json
clangine uninstall # remove hooks from settings
```

## Backends

### ffplay (default)

Plays local audio files from `tracks/`. Requires ffmpeg.

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg
```

Drop .mp3/.wav/.ogg/.flac/.m4a files into `tracks/` and go.

### Spotify (macOS)

Controls the Spotify desktop app via AppleScript. No local files needed.

```bash
# config.sh
CLANGINE_BACKEND="spotify"
CLANGINE_PLAYLIST="spotify:playlist:37i9dQZF1DX..."  # optional
CLANGINE_SHUFFLE="true"
```

To get a playlist URI: Spotify → right-click playlist → Share → Copy Spotify URI.

If no playlist is set, it resumes whatever was last playing. If Spotify isn't open, clangine launches it automatically.

## Configuration

Create `config.sh` in the clangine directory:

```bash
# Backend: "ffplay" (local files) or "spotify" (macOS Spotify app)
CLANGINE_BACKEND="ffplay"

# ffplay settings
CLANGINE_VOLUME="0.3"        # 0.0 to 1.0

# Spotify settings
CLANGINE_PLAYLIST=""          # spotify:playlist:xxx or spotify:album:xxx

# Shared
CLANGINE_SHUFFLE="true"      # randomize track order
CLANGINE_LOOP="true"         # loop playlist (ffplay only)
```

## How it works (technical)

**ffplay backend:**
- Manages an `ffplay` process in the background
- PID tracked in `/tmp/clangine-de-poitrine.pid`
- State (playing/paused/stopped) in `/tmp/clangine-de-poitrine.state`
- Pause/resume via SIGSTOP/SIGCONT signals (no restart, no seek loss)
- Track queue shuffled on startup, advances on finish or skip

**Spotify backend:**
- Controls Spotify via `osascript` (AppleScript)
- Play/pause/skip map directly to Spotify commands
- Auto-launches Spotify if not running
- Can load a specific playlist on first play

Hooks are simple one-liners in `~/.claude/settings.json` that call `clangine play` or `clangine pause`.

---

*Named after [Angine de Poitrine](https://anginedepoitrine.bandcamp.com/) — microtonal math rock from Saguenay, QC. The perfect coding soundtrack.*

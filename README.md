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

## Requirements

- `ffplay` (comes with ffmpeg) — the audio backend
- Claude Code CLI

### Install ffmpeg

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt install ffmpeg

# or any other method — we just need ffplay
```

## Configuration

Edit `~/.clangine-de-poitrine/config.sh` (created on first run):

```bash
JUKEBOX_PLAYER="ffplay"     # audio player command
JUKEBOX_VOLUME="0.3"        # 0.0 to 1.0
JUKEBOX_SHUFFLE="true"      # randomize track order
JUKEBOX_LOOP="true"         # loop playlist
```

## Adding tracks

Drop files into `tracks/`:
```
tracks/
  angine-vol1-01.mp3
  angine-vol2-03.mp3
  whatever-you-want.ogg
```

Supports: .mp3, .wav, .ogg, .flac, .m4a

## How it works (technical)

The `clangine` script manages an `ffplay` process in the background:
- PID tracked in `/tmp/clangine-de-poitrine.pid`
- State (playing/paused/stopped) in `/tmp/clangine-de-poitrine.state`
- Pause/resume via SIGSTOP/SIGCONT signals (no restart, no seek loss)
- Track queue shuffled on startup, advances on finish or skip

Hooks are simple one-liners in `~/.claude/settings.json` that call `clangine play` or `clangine pause`.

---

*Named after [Angine de Poitrine](https://anginedepoitrine.bandcamp.com/) — microtonal math rock from Saguenay, QC. The perfect coding soundtrack.*

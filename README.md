# clangine-de-poitrine  ✋🔼🤚

Claude code plays [Angine de Poitrine — Vol. II](https://anginedepoitrine.bandcamp.com/album/vol-ii) on Spotify as long as it's running. 
When the music stops, it means you need to do something (accept permissions, continue prompting, etc)

<video src="assets/demo.mp4" width="100%"></video>

## Quick start

```bash
# 1. Install
git clone https://github.com/jerpint/clangine-de-poitrine.git ~/.clangine-de-poitrine
cd ~/.clangine-de-poitrine
chmod +x clangine

# 2. Install hooks into Claude Code
./clangine install

# 3. That's it — next time you use Claude Code, music plays while it thinks
```


## How it works

Claude Code hooks trigger shell scripts:
- `UserPromptSubmit` → `clangine play` (you hit enter, music starts)
- `PostToolUse` → `clangine play` (resumes after permission was granted)
- `Stop` → `clangine pause` (Claude finishes, music stops)
- `Notification` → `clangine pause` (Claude needs your attention)
- `PermissionRequest` → `clangine pause` (Claude needs permission)

Controls the Spotify desktop app via AppleScript (macOS).


## Configuration

Create `config.sh` in the clangine directory:

```bash
CLANGINE_PLAYLIST="spotify:playlist:37i9dQZF1DX..."  # playlist or album URI
CLANGINE_SHUFFLE="true"                               # randomize track order
```

To get a playlist URI: Spotify → right-click playlist → Share → Copy Spotify URI.

If no playlist is set, it resumes whatever was last playing. If Spotify isn't open, clangine launches it automatically.

By default it plays [Angine de Poitrine — Vol. II](https://anginedepoitrine.bandcamp.com/).

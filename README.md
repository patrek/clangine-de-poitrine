# clangine-de-poitrine  ✋🔼🤚

Claude code plays [Angine de Poitrine — Vol. II](https://anginedepoitrine.bandcamp.com/album/vol-ii) on Spotify as long as it's running. 
When the music stops, it means you need to do something (accept permissions, continue prompting, etc)

https://github.com/user-attachments/assets/c0d2b255-a814-4320-a967-1ade247c39c3


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

Controls the Spotify desktop app:
- **macOS** — via AppleScript (built in, no extra dependencies)
- **Linux** — via [`playerctl`](https://github.com/altdesktop/playerctl) over MPRIS/D-Bus (works on KDE, GNOME, etc.)

On Linux, install `playerctl` first:

```bash
sudo apt install playerctl     # Debian/Ubuntu
sudo pacman -S playerctl       # Arch
sudo dnf install playerctl     # Fedora
```


## Configuration

Create `config.sh` in the clangine directory:

```bash
CLANGINE_PLAYLIST="spotify:playlist:37i9dQZF1DX..."  # playlist or album URI
CLANGINE_SHUFFLE="true"                               # randomize track order
CLANGINE_PAUSE_ON="questions"                         # "stop" (default): pause every turn end
                                                      # "questions": only pause when Claude needs input
```

To get a playlist URI: Spotify → right-click playlist → Share → Copy Spotify URI.

If no playlist is set, it resumes whatever was last playing. Only an explicit `clangine on` / `clangine play` will launch Spotify if it isn't open — the automatic hooks never start Spotify on their own. While Claude works, clangine only resumes a playback it paused itself; if Spotify is closed or you're playing music manually, it stays out of the way.

By default it plays [Angine de Poitrine — Vol. II](https://anginedepoitrine.bandcamp.com/).

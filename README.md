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

On Linux, install `playerctl` first (and `jq` for `./clangine install`):

```bash
sudo apt install playerctl jq     # Debian/Ubuntu
sudo pacman -S playerctl jq       # Arch
sudo dnf install playerctl jq     # Fedora
```

The `.deb`, Snap and Flatpak (`com.spotify.Client`) builds of Spotify are all supported. Hooks must be able to reach your session D-Bus, so Claude Code needs to run inside your desktop session (not over plain SSH or in a container without the bus).

Troubleshooting: `playerctl -l` should list `spotify` while Spotify is open, and `./clangine status` shows what clangine sees.


## Configuration

Create `config.sh` in the clangine directory:

```bash
CLANGINE_PLAYLIST="spotify:playlist:37i9dQZF1DX..."  # playlist or album URI
CLANGINE_SHUFFLE="true"                               # randomize track order
CLANGINE_PAUSE_ON="questions"                         # "stop" (default): pause every turn end
                                                      # "questions": only pause when Claude needs input
CLANGINE_PLAYER="spotifyd"                            # Linux only: MPRIS player to drive (default: spotify)
```

`CLANGINE_PLAYER` lets you drive another MPRIS client (spotifyd, ncspot, spotify-player…), using the name shown by `playerctl -l`. clangine only launches the official Spotify app, so start other players yourself.

To get a playlist URI: Spotify → right-click playlist → Share → Copy Spotify URI.

If no playlist is set, it resumes whatever was last playing. Only an explicit `clangine on` / `clangine play` will launch Spotify or switch to the playlist — the automatic hooks never do. They do start playback on their own when clangine is on and Spotify is open but idle, so you can turn clangine on (or open Spotify) while Claude is already working and the music kicks in at its next step.

clangine stays out of your way otherwise: music you're already playing yourself is left alone, and if you pause during a turn, the hooks won't restart it until you press play again. `./clangine install` and `uninstall` only touch clangine's own hooks — any other hooks in `settings.json` are kept.

By default it plays [Angine de Poitrine — Vol. II](https://anginedepoitrine.bandcamp.com/).

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, Gemini CLI, etc.) when working with code in this repository.

## What this is

clangine-de-poitrine plays Spotify while Claude Code works, via Claude Code hooks: music plays while Claude is thinking, pauses when it needs user input. The entire implementation is the single bash script `clangine`.

## Commands

There is no build or test suite. Verify changes with:

```bash
bash -n clangine          # syntax check
shellcheck clangine       # lint (if installed)
./clangine status         # exercise the script manually
```

Manual commands: `on` / `off` / `toggle` (activation), `play` / `pause` / `stop` / `skip`, `status`, `install` / `uninstall` (writes/removes hooks in `~/.claude/settings.json` using jq).

## Architecture

Everything lives in `clangine`, organized in layers (top to bottom in the file):

- **Config**: defaults in the script, overridden by sourcing `config.sh` (gitignored, user-local). Key vars: `CLANGINE_PLAYLIST`, `CLANGINE_SHUFFLE`, `CLANGINE_PAUSE_ON` (`stop` = pause every turn end, `questions` = only pause when Claude waits for input), `CLANGINE_PLAYER` (Linux MPRIS player name, default `spotify`).
- **State**: `$XDG_RUNTIME_DIR/clangine-de-poitrine.state` (falls back to `$TMPDIR`, then `/tmp`) holds `state:album` (e.g. `playing:Vol. II`). States: `playing` (clangine is driving), `paused` (clangine paused it — hooks resume if still on the same album), `held` (user paused by hand, or `stop` — hooks stay quiet until the user presses play again, then follow along), `stopped` (not driving — hooks may start an idle, already-open player). `cmd_pause` checks the real player status so a manual pause becomes `held`, not `paused`. Activation is the presence of the `.enabled` file (gitignored).
- **Player abstraction**: `_player_*` functions dispatch on `CLANGINE_OS` — macOS uses AppleScript (`_osa`), Linux uses `playerctl -p "$CLANGINE_PLAYER"` over MPRIS/D-Bus (`_pctl`). Any new player operation must implement both branches. On macOS, guard with `_player_running` first: any `tell application "Spotify"` launches the app. Linux quirks: `_player_running` asks MPRIS (not `pgrep`) because Spotify's process appears seconds before it's on D-Bus, so launch is followed by `_player_wait_ready` polling; `playerctl open` is async, so `_player_wait_track_change` polls the track id before reading metadata; Spotify may reject MPRIS shuffle, which is non-fatal.
- **Commands**: `cmd_*` functions. The critical distinction is `cmd_play` (explicit: may launch Spotify and start the playlist) vs `cmd_resume` (hook-driven: never launches Spotify, never opens the playlist; resumes `paused`, follows the user out of `held`, and starts an idle open player from `stopped`). `cmd_resume` runs on every PostToolUse, so it reads the state file before touching the player, and the payload predicates use bash builtins rather than `grep`.
- **Errors under `set -e`**: a failing `playerctl` call (e.g. "No players found") aborts the script before the state file is written, so player calls whose failure is expected must be guarded. From a hook (`CLANGINE_HOOK=1`), a missing `playerctl` exits 0 silently.
- **Hook dispatch** (bottom `case` block): hooks pass a JSON payload on stdin; interactive invocations don't. `_read_payload` consumes stdin once and the result is passed to predicates:
  - `_from_hook` routes `play` to passive `cmd_resume` instead of `cmd_play`.
  - `_is_self_trigger` skips PostToolUse hooks fired by clangine's own Bash invocations (avoids feedback loops).
  - `_skip_turn_end_pause` implements `CLANGINE_PAUSE_ON=questions` by ignoring `Stop`-event pauses.
- **Install**: `cmd_install`/`cmd_uninstall` use the `strip_clangine` jq filter to remove only hook commands starting with this script's path, so other hooks on the same events survive and re-installing doesn't duplicate entries.

Installed hook wiring: `UserPromptSubmit`/`PostToolUse` → `play`; `Stop`/`Notification`/`PermissionRequest` → `pause`.

## Working in this repo

This repo at `~/.clangine-de-poitrine` **is the live installation** — the hooks in `~/.claude/settings.json` point at this very script, and they fire during your own session. Edits take effect immediately on the next hook event, so a broken edit can pause/launch Spotify or spam errors mid-session. The `_is_self_trigger` guard is what prevents your own `./clangine` test invocations from re-triggering playback via PostToolUse — preserve it when touching the dispatch block.

`config.sh` and `.enabled` are gitignored user-local files: never commit them, and don't assume `config.sh` exists.

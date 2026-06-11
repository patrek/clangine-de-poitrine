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

- **Config**: defaults in the script, overridden by sourcing `config.sh` (gitignored, user-local). Key vars: `CLANGINE_PLAYLIST`, `CLANGINE_SHUFFLE`, `CLANGINE_PAUSE_ON` (`stop` = pause every turn end, `questions` = only pause when Claude waits for input).
- **State**: `/tmp/clangine-de-poitrine.state` holds `state:album` (e.g. `playing:Vol. II`). The album is saved so clangine only resumes playback *it* paused — if the user switched albums or plays music manually, it stays out of the way. Activation is the presence of the `.enabled` file (gitignored).
- **Player abstraction**: `_player_*` functions dispatch on `CLANGINE_OS` — macOS uses AppleScript (`_osa`), Linux uses `playerctl` over MPRIS/D-Bus. Any new player operation must implement both branches. Linux quirk: `playerctl open` is async, hence the `sleep 1` before reading metadata.
- **Commands**: `cmd_*` functions. The critical distinction is `cmd_play` (explicit: may launch Spotify and start the playlist) vs `cmd_resume` (hook-driven: passive, never launches Spotify, never starts a playlist, only resumes a clangine-paused playback).
- **Hook dispatch** (bottom `case` block): hooks pass a JSON payload on stdin; interactive invocations don't. `_read_payload` consumes stdin once and the result is passed to predicates:
  - `_from_hook` routes `play` to passive `cmd_resume` instead of `cmd_play`.
  - `_is_self_trigger` skips PostToolUse hooks fired by clangine's own Bash invocations (avoids feedback loops).
  - `_skip_turn_end_pause` implements `CLANGINE_PAUSE_ON=questions` by ignoring `Stop`-event pauses.

Installed hook wiring: `UserPromptSubmit`/`PostToolUse` → `play`; `Stop`/`Notification`/`PermissionRequest` → `pause`.

## Working in this repo

This repo at `~/.clangine-de-poitrine` **is the live installation** — the hooks in `~/.claude/settings.json` point at this very script, and they fire during your own session. Edits take effect immediately on the next hook event, so a broken edit can pause/launch Spotify or spam errors mid-session. The `_is_self_trigger` guard is what prevents your own `./clangine` test invocations from re-triggering playback via PostToolUse — preserve it when touching the dispatch block.

`config.sh` and `.enabled` are gitignored user-local files: never commit them, and don't assume `config.sh` exists.

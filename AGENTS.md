# AGENTS.md

Instructions for agents working in this repository.

## What this repo is

A pair of keybinding configs that make editing inside **OpenCode** behave like
editing in **Windows Notepad** (`Ctrl+C`/`Ctrl+V` copy-paste, `Ctrl+Z`/`Ctrl+Y`
undo-redo, `Ctrl+A` select all, `Home`/`End`, `Shift+Enter` for a newline).

## The two files, and why both matter

| File | Applies to | Role |
| --- | --- | --- |
| `opencode/cli.json` | OpenCode CLI | **What the keys do.** Maps key combinations to OpenCode input/session actions. |
| `windows-terminal/keybindings.json` | Windows Terminal | **Whether the keys arrive at all.** Intercepts `Ctrl+C`/`Ctrl+V` for the clipboard and forwards `Shift+Enter`, `Ctrl+Backspace`, `Ctrl+X` as escape sequences. |

**Both must be applied.** Applying only one produces a broken mix:

- OpenCode alone: Windows Terminal still turns `Ctrl+C` into SIGINT/copy at the
  terminal layer, and never sends `Shift+Enter` as a distinct key, so OpenCode's
  `input.newline` binding never fires and `Shift+Enter` submits instead.
- Windows Terminal alone: `Ctrl+C` copies but OpenCode still treats it as an
  interrupt, and the other bindings have nothing to trigger.

When changing a key, change it in **both** files (if Windows Terminal has to
forward it) and say so in the PR/commit.

## Installation

### 1. OpenCode — `opencode/cli.json`

Copy the file to the OpenCode CLI config path:

- Linux / WSL / macOS: `~/.config/opencode/cli.json`
- Windows: `%USERPROFILE%\.config\opencode\cli.json`

If the target already exists, merge the `keybinds` object into it — do not blindly
overwrite unrelated settings (theme, diffs, session, animations). OpenCode reads
this config at startup, so restart the TUI after changing it.

### 2. Windows Terminal — `windows-terminal/keybindings.json`

This file is a **fragment**, not a complete `settings.json`. Merge it into the
user's settings file:

1. Append every entry of the fragment's `actions` array to the top-level
   `"actions"` array (create the array if missing).
2. Append every entry of the fragment's `keybindings` array to the top-level
   `"keybindings"` array.

Settings file locations:

- Microsoft Store build:
  `%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json`
- Preview (Store) build:
  `%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminalPreview_8wekyb3d8bbwe\LocalState\settings.json`
- Unpackaged / winget / Scoop:
  `%LOCALAPPDATA%\Microsoft\Windows Terminal\settings.json`

Never replace the whole `settings.json` with the fragment: that would destroy the
user's profiles, GUIDs, colour schemes and startup directory. Merge only.

Windows Terminal hot-reloads the file on save, so no restart is needed.

## Verifying a change

1. OpenCode: `Ctrl+Z` undoes instead of suspending, `Ctrl+C` copies, `Shift+Enter`
   inserts a newline, `Enter` submits.
2. Windows Terminal: selecting text and pressing `Ctrl+C` copies it; `Ctrl+V`
   pastes plain text.
3. Both files remain valid JSON (`python3 -m json.tool <file>`).

## Conventions

- Key syntax follows OpenCode's keybind format: modifiers in the order
  `ctrl`/`alt`/`shift` + key name, joined by `+` (for example `ctrl+shift+k`).
  Use `none` to unbind an action.
- Windows Terminal `sendInput` sequences use CSI-u encoding:
  `\u001b[<codepoint>;<modifier>u`, where the modifier value is `1 + shift(1) +
  alt(2) + ctrl(4)`. For example `Ctrl+Backspace` → `\u001b[127;5u` and
  `Shift+Enter` → `\u001b[13;2u`.
- Keep the README key tables in sync with the JSON files. The README is the user
  facing documentation; this file is the operational one.
- Do not add non-keybinding settings to `windows-terminal/keybindings.json`
  beyond what is needed for clipboard/typing behaviour.

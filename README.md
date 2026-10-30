# open-code-keybindings

Keybindings that make editing inside [OpenCode](https://opencode.ai) feel like
editing in **Windows Notepad**: `Ctrl+C`/`Ctrl+V` copy and paste, `Ctrl+Z`/`Ctrl+Y`
undo and redo, `Ctrl+A` selects everything, `Home`/`End` jump to the edges of the
line, `Shift+Enter` inserts a newline, and `Enter` sends.

Out of the box OpenCode behaves like a terminal application, not like a text
editor: `Ctrl+C` interrupts the running session, `Shift+Enter` submits, and
`Ctrl+Z` suspends the process. These files remap that behaviour to the familiar
Windows shortcuts.

## Contents

| Path | What it is |
| --- | --- |
| `opencode/cli.json` | OpenCode CLI configuration. The `keybinds` block is what this repo is about. Goes to `~/.config/opencode/cli.json`. |
| `windows-terminal/keybindings.json` | Fragment to merge into Windows Terminal's `settings.json` — it holds the `actions` and `keybindings` arrays. |
| `AGENTS.md` | Notes for agents/LLMs working in this repo: which file is which, and that **both** must be applied. |

## Why two files?

Both sides are needed, and they only work together:

- **Windows Terminal** decides what the terminal emulator does with a key before
  OpenCode ever sees it. It intercepts `Ctrl+C`/`Ctrl+V` for clipboard work and
  must explicitly forward `Shift+Enter`, `Ctrl+Backspace` and `Ctrl+X` as escape
  sequences using `sendInput` — otherwise those combinations are swallowed or
  translated into something else.
- **OpenCode** decides what to do with the keys it receives: copy the message,
  undo the input, delete the previous word, insert a newline instead of
  submitting, and so on.

Applying only one half gives you a mismatch: OpenCode waiting for a key Windows
Terminal never sends, or Windows Terminal copying on `Ctrl+C` while OpenCode
still treats it as an interrupt.

## OpenCode keybindings

| Key | Action | Notepad-like behaviour |
| --- | --- | --- |
| `Ctrl+C` | `messages.copy` | Copies instead of interrupting the session |
| `Ctrl+V` | paste | Paste into the prompt |
| `Ctrl+X` | pass-through | Forwarded by Windows Terminal (`session.export` is disabled so the key is free) |
| `Ctrl+A` | `input.select.all` | Select all |
| `Ctrl+Z` | `input.undo` | Undo |
| `Ctrl+Y` | `input.redo` | Redo |
| `Ctrl+Backspace` | `input.delete.word.backward` | Delete the previous word |
| `Ctrl+Home` / `Ctrl+End` | `input.line.home` / `input.line.end` | Jump to the start/end of the line |
| `Home` / `End` | `input.visual.line.home` / `input.visual.line.end` | Jump to the start/end of the visible line |
| `Shift+Enter` | `input.newline` | Newline instead of submit; plain `Enter` still sends |
| `Esc` | `prompt.clear` | Clear the prompt |
| `Alt+Home` / `Alt+End` | `session.first` / `session.last` | Jump to the first/last message |
| `<leader><leader>` | `app.exit` | Quit (the leader key must be pressed twice) |

Two defaults are switched off on purpose:

- `terminal.suspend` → `none`, freeing `Ctrl+Z` for undo (otherwise OpenCode
  suspends the whole TUI).
- `session.export` → `none`, freeing `Ctrl+X` for the terminal pass-through.

## Windows Terminal keybindings

| Key | Action | Why |
| --- | --- | --- |
| `Ctrl+C` | `Terminal.CopyToClipboard` | Copy selected text instead of sending SIGINT |
| `Ctrl+V` | `Terminal.PasteFromClipboard` | Paste |
| `Shift+Enter` | `User.sendInput.ShiftEnter` | Sends `CSI 13;2u` so OpenCode inserts a newline |
| `Ctrl+Backspace` | `User.sendInput.CtrlBackspace` | Sends `CSI 127;5u` so OpenCode deletes a word |
| `Ctrl+X` | `User.sendInput.CtrlX` | Sends `CSI 120;5u` so OpenCode receives the modified key |
| `Alt+Shift+D` | `Terminal.DuplicatePaneAuto` | Duplicate the current pane (unrelated convenience) |

## Install

### 1. OpenCode

Copy `opencode/cli.json` to the OpenCode CLI config location:

- **Linux / WSL / macOS:** `~/.config/opencode/cli.json`
- **Windows:** `%USERPROFILE%\.config\opencode\cli.json`

If you already have a `cli.json`, merge only its `keybinds` object into yours
instead of replacing the file. Restart OpenCode afterwards.

### 2. Windows Terminal

Open Windows Terminal → **Settings** → **Open JSON file**, then:

1. Merge the entries from `actions` into the top-level `"actions"` array.
2. Merge the entries from `keybindings` into the top-level `"keybindings"` array.

Do **not** replace your whole `settings.json` with `windows-terminal/keybindings.json`
— it is a fragment, and replacing the file would wipe your profiles, colour
schemes and startup settings. Save, and Windows Terminal reloads automatically.

For the most Notepad-like behaviour you may also want these top-level settings:

```json
"copyOnSelect": false,
"copyFormatting": "none"
```

`copyFormatting: "none"` makes `Ctrl+V` paste plain text instead of rich text
with colours and fonts, which is what a text editor does.

### Settings file locations

| Install type | Path |
| --- | --- |
| Microsoft Store | `%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json` |
| Unpackaged / Scoop / winget | `%LOCALAPPDATA%\Microsoft\Windows Terminal\settings.json` |
| Preview (Store) | `%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminalPreview_8wekyb3d8bbwe\LocalState\settings.json` |

## Caveats

- `Ctrl+C` is a clipboard key inside Windows Terminal, so it will not send an
  interrupt to OpenCode or to programs running in a shell pane of that profile.
  Use another profile/tab if you need SIGINT.
- `Ctrl+Shift+C` / `Ctrl+Shift+V` remain available as the shell-style
  copy/paste pair if you prefer them.
- `Shift+Enter` only becomes a newline when the receiving application understands
  the `CSI 13;2u` sequence. OpenCode does; some other REPLs may not.

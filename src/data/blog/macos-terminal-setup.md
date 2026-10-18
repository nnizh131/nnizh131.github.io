---
title: "My macOS terminal setup: Ghostty, tmux, and workmux"
author: Nika Nizharadze
pubDatetime: 2026-10-18T08:00:00Z
modDatetime: 2026-10-18T12:00:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "A practical terminal setup for coding agents, with Rosé Pine Dawn, reliable copying, and a workmux sidebar that actually shows status."
---

I wanted a terminal where I could keep a coding task running, move between branches, and see whether an agent was working or waiting for me. I also wanted selecting text and copying it to work reliably. That last part took more effort than I expected.

The setup I landed on is Ghostty, Zsh, Powerlevel10k, tmux, and workmux. The palette is Rosé Pine Dawn, with JetBrains Mono for the font. This is a record of what I configured and the things that broke along the way. A few choices, including my tmux prefix, are still open.

## What each piece does

Ghostty is the terminal application. It handles the font, colors, native selection, and macOS shortcuts. Zsh runs the shell, and Powerlevel10k builds the prompt: the directory, Git branch, status, and icons.

tmux holds the sessions, windows, and panes. I can detach and reconnect while the server keeps running. That doesn't mean the session survives a reboot; I haven't added session restoration.

[workmux](https://github.com/raine/workmux) connects Git worktrees to terminal windows and provides an agent dashboard and sidebar. I use it to keep a task's checkout and terminal together.

Separating those responsibilities helped with debugging. Changing the terminal theme didn't fix the prompt's colors, and fixing terminal selection didn't automatically fix copying inside Codex.

## Install the basics

On macOS, with Homebrew already installed:

```sh
brew install --cask ghostty
brew install tmux powerlevel10k jq
brew install raine/workmux/workmux
```

Ghostty already bundles JetBrains Mono and Nerd Font symbols, so no separate font installation is needed for this terminal setup. The Homebrew font cask is only useful if you also want the font available to other applications. See [Ghostty's bundled defaults](https://ghostty.org/docs/config).

I ran the installs sequentially and backed up the existing configuration files before editing them. I kept unrelated settings rather than replacing whole files with a new set of dotfiles.

The commands below run in a terminal shell unless explicitly marked as a command to enter inside Codex. I use `nano` to edit configuration files: paste the indicated settings, press **Ctrl+O**, then **Enter** to save, and **Ctrl+X** to exit. If a setting already exists, change that line rather than adding a duplicate. Backups use a timestamp so repeated edits keep the earlier versions.

Follow the sections in order. The configuration snippets belong in the file opened immediately above them; don't execute INI, TOML, YAML, or tmux settings as shell commands. Rerunning the steps should update existing settings, not append another copy. The intended result is Dawn colors, the segmented prompt, working clipboard support, a left workmux sidebar, and separate agent footers. The final section lists the checks that confirm that result.

## A light terminal and a familiar prompt

My active Ghostty config is under `~/Library/Application Support/com.mitchellh.ghostty/config`. Check the active path through Ghostty's settings before editing; another installation may use `~/.config/ghostty/config`.

For the path on my Mac, run these commands to create the directory if needed, back up an existing config, and open it:

```sh
ghostty_config="$HOME/Library/Application Support/com.mitchellh.ghostty/config"
mkdir -p "$(dirname "$ghostty_config")"
if [ -f "$ghostty_config" ]; then
  cp -p "$ghostty_config" "$ghostty_config.backup-$(date +%Y%m%d-%H%M%S)"
fi
nano "$ghostty_config"
```

If Ghostty reports the other path, use this block instead:

```sh
ghostty_config="$HOME/.config/ghostty/config"
mkdir -p "$(dirname "$ghostty_config")"
if [ -f "$ghostty_config" ]; then
  cp -p "$ghostty_config" "$ghostty_config.backup-$(date +%Y%m%d-%H%M%S)"
fi
nano "$ghostty_config"
```

Paste or update these appearance settings in the file:

```ini
font-family = JetBrains Mono
font-size = 14
theme = Rose Pine Dawn
window-padding-x = 10
window-padding-y = 8
macos-option-as-alt = true
```

Save with **Ctrl+O**, **Enter**, then exit with **Ctrl+X**. Validate the configuration and inspect the effective appearance settings:

```sh
/Applications/Ghostty.app/Contents/MacOS/ghostty +validate-config
/Applications/Ghostty.app/Contents/MacOS/ghostty +show-config |
  rg '^(font-family|font-size|theme|background) ='
```

The second command uses `rg`; install it with `brew install ripgrep` if needed. Reload Ghostty with **Cmd+Shift+,** after validation succeeds.

Size 14 is what I'm trying. Dawn's background is a very pale cream, `#faf4ed`. Ghostty supplies both JetBrains Mono and the symbol fallback for the icons in my prompt. The explicit `font-family` line documents the choice; it doesn't require a separate font install.

I kept the existing single-line, segmented Powerlevel10k layout. It already showed the things I wanted: macOS, current directory, Git branch, and command status. I changed its fixed colors to fit Dawn:

First, back up and open the shell configuration:

```sh
if [ -f ~/.zshrc ]; then
  cp -p ~/.zshrc ~/.zshrc.backup-$(date +%Y%m%d-%H%M%S)
fi
nano ~/.zshrc
```

Add the following once. If another prompt theme is loaded, replace its loading line. Keep unrelated shell settings:

```sh
source "$(brew --prefix)/share/powerlevel10k/powerlevel10k.zsh-theme"
[[ ! -f ~/.p10k.zsh ]] || source ~/.p10k.zsh
alias mux='tmux new-session -A -s work'
alias wm='workmux'
```

Save and exit. If you don't have a Powerlevel10k configuration yet, start from its installed template:

```sh
if [ ! -f ~/.p10k.zsh ]; then
  cp "$(brew --prefix powerlevel10k)/config/p10k-classic.zsh" ~/.p10k.zsh
fi
cp -p ~/.p10k.zsh ~/.p10k.zsh.backup-$(date +%Y%m%d-%H%M%S)
nano ~/.p10k.zsh
```

In that file, update the matching settings to these values, then save and exit. Preserve existing prompt-element arrays if you already have a layout you like:

```zsh
typeset -g POWERLEVEL9K_BACKGROUND='#f2e9e1'
typeset -g POWERLEVEL9K_OS_ICON_FOREGROUND='#907aa9'
typeset -g POWERLEVEL9K_DIR_FOREGROUND='#286983'
typeset -g POWERLEVEL9K_DIR_SHORTENED_FOREGROUND='#575279'
typeset -g POWERLEVEL9K_DIR_ANCHOR_FOREGROUND='#286983'
typeset -g POWERLEVEL9K_VCS_CLEAN_FOREGROUND='#286983'
typeset -g POWERLEVEL9K_VCS_UNTRACKED_FOREGROUND='#907aa9'
typeset -g POWERLEVEL9K_VCS_MODIFIED_FOREGROUND='#b4637a'
typeset -g POWERLEVEL9K_VCS_VISUAL_IDENTIFIER_COLOR='#286983'
typeset -g POWERLEVEL9K_VCS_LOADING_VISUAL_IDENTIFIER_COLOR='#797593'
```

For a fresh template, replace the complete left and right prompt-element arrays, including their old closing parentheses, with:

```zsh
typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(os_icon dir vcs)
typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(status command_execution_time background_jobs)
typeset -g POWERLEVEL9K_PROMPT_ADD_NEWLINE=false
```

The palette I use is:

| Prompt element                | Color     |
| ----------------------------- | --------- |
| Segment background            | `#f2e9e1` |
| Main text                     | `#575279` |
| Directory and clean Git state | `#286983` |
| OS icon and untracked count   | `#907aa9` |
| Modifications and errors      | `#b4637a` |

The classic template's custom Git formatter has its own color escapes. To update those and the remaining numeric foreground settings, install Python if needed with `brew install python`, then paste this entire command into the shell. It validates the generated Zsh before replacing the file and creates another backup:

```sh
python3 <<'PY'
from datetime import datetime
from pathlib import Path
import re
import shutil
import subprocess
import tempfile

path = Path.home() / ".p10k.zsh"
text = path.read_text()
text = re.sub(
    r"(?m)^(\s*typeset -g POWERLEVEL9K_\S*FOREGROUND=)\d+$",
    r"\1'#575279'", text,
)
for old, color in {
    248: "575279", 76: "286983", 178: "b4637a",
    39: "907aa9", 196: "b4637a", 244: "797593", 246: "797593",
}.items():
    text = text.replace(f"%{old}F", f"%F{{#{color}}}")
text = re.sub(r"%#([0-9a-fA-F]{6})F", r"%F{#\1}", text)
text = re.sub(
    r"(?m)^(\s*typeset -g POWERLEVEL9K_\S*ERROR\S*FOREGROUND=).*$",
    r"\1'#b4637a'", text,
)
with tempfile.NamedTemporaryFile(mode="w", suffix=".zsh") as candidate:
    candidate.write(text)
    candidate.flush()
    subprocess.run(["zsh", "-n", candidate.name], check=True)
backup = path.with_name(path.name + ".backup-" + datetime.now().strftime("%Y%m%d-%H%M%S-%f"))
shutil.copy2(path, backup)
path.write_text(text)
print(f"Updated {path}; backup: {backup}")
PY
```

This targets the classic template; inspect other custom formatters before adapting it. One broken attempt printed a color escape as literal text; the valid Zsh form is `%F{#797593}`, followed by `%f` to reset.

Editing the prompt arrays also caught me out. A replacement stopped at a parenthesis inside a comment and left old entries behind. Now I check the whole array and run:

```sh
zsh -n ~/.zshrc
zsh -n ~/.p10k.zsh
```

If both syntax checks pass, reload in the interactive shell:

```sh
source ~/.zshrc
source ~/.p10k.zsh
```

Ghostty settings reload with `Cmd+Shift+,`.

## Start tmux and configure the base settings

Back up and open the tmux configuration:

```sh
if [ -f ~/.tmux.conf ]; then
  cp -p ~/.tmux.conf ~/.tmux.conf.backup-$(date +%Y%m%d-%H%M%S)
fi
nano ~/.tmux.conf
```

Paste or update these base settings, then save and exit. Keep existing bindings and the current prefix:

```text
set -g mouse on
set -g history-limit 100000
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
set -sg escape-time 10
set -g focus-events on
set -g default-terminal 'tmux-256color'
set -as terminal-features ',xterm-ghostty:RGB:clipboard'
set -s set-clipboard on
setw -g mode-keys vi
```

From a shell outside tmux, create or attach to the work session:

```sh
tmux new-session -A -s work
```

From inside that session, apply the file and inspect the clipboard settings:

```sh
tmux source-file ~/.tmux.conf
tmux show-options -g mouse
tmux show-options -s set-clipboard
```

Expected output includes `mouse on` and `set-clipboard on`. Existing panes may need to be recreated for terminal-type and history-limit changes to take effect. Keep running tasks intact; don't kill the server to reload configuration.

## Getting copying to work

There were three places to check: Ghostty, tmux, and the agent interface.

Reopen the active Ghostty config using the appropriate path above:

```sh
nano "$ghostty_config"
```

If this is a new shell, set `ghostty_config` again using the matching earlier block. Paste or update these clipboard settings, save and exit, then reload Ghostty with **Cmd+Shift+,**:

```ini
mouse-shift-capture = never
clipboard-write = allow
clipboard-trim-trailing-spaces = true
copy-on-select = clipboard
keybind = super+c=copy_to_clipboard
keybind = super+v=paste_from_clipboard
```

Holding Shift while dragging lets me make a native terminal selection when an application captures the mouse. The exact options are documented in [Ghostty's configuration reference](https://ghostty.org/docs/config/reference).

Inside tmux, I needed mouse support and clipboard bindings for both copy-mode tables. Back up and open the file again:

```sh
cp -p ~/.tmux.conf ~/.tmux.conf.backup-$(date +%Y%m%d-%H%M%S)
nano ~/.tmux.conf
```

Add the bindings below; the first three settings are already in the base configuration, so update their existing lines rather than duplicating them:

```text
set -g mouse on
set -s set-clipboard on
setw -g mode-keys vi

bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
```

Save and exit, then reload and check both mouse-release bindings:

```sh
tmux source-file ~/.tmux.conf
tmux list-keys -T copy-mode MouseDragEnd1Pane
tmux list-keys -T copy-mode-vi MouseDragEnd1Pane
```

Select a harmless word, then run this in another shell to check the clipboard contents:

```sh
pbpaste
```

The earlier configuration only covered Vi copy mode, while the running server was using Emacs copy mode. Covering both fixed that gap.

`set-clipboard on` also allows applications inside tmux to write through OSC 52. The `external` setting blocks application clipboard requests. That distinction mattered for Codex; see the [tmux clipboard guide](https://github.com/tmux/tmux/wiki/Clipboard). The `pbcopy` bindings above target a local macOS session.

For Codex CLI 0.163.0, open its configuration from a shell:

```sh
codex_config_dir="${CODEX_HOME:-$HOME/.codex}"
mkdir -p "$codex_config_dir"
if [ -f "$codex_config_dir/config.toml" ]; then
  cp -p "$codex_config_dir/config.toml" "$codex_config_dir/config.toml.backup-$(date +%Y%m%d-%H%M%S)"
fi
nano "$codex_config_dir/config.toml"
```

Add `copy_on_select` under the existing `[tui]` heading, or create that table if absent. Don't add a second `[tui]` table:

```toml
[tui]
copy_on_select = "always"
```

Save and exit. Finish any running turn and quit Codex, then resume the intended conversation from the same project directory:

```sh
codex resume
```

I restarted Codex and tested selecting a response and pasting into another window. Copying now works in my direct Codex session and inside tmux. I still need to test the separate Claude Code clipboard path.

## The sidebar needs a registered agent

The workmux sidebar initially said **No agents running**, even though Codex was open. The pane hadn't been registered with workmux. Registering the actual Codex pane made the entry appear, with its working status.

That gives me a useful debugging order: check `workmux status` before debugging the sidebar itself. An empty registry explains an empty sidebar.

From a shell inside tmux, inspect the worktrees and tracked agents:

```sh
workmux list
workmux status
workmux dashboard
```

Press `q` to leave the dashboard. To create a new task checkout deliberately, run `workmux add my-feature`; to reopen an existing one, use `workmux open my-feature`.

To recover an already-running Codex pane that wasn't registered, first list the pane IDs:

```sh
tmux list-panes -a -F '#{pane_id} #{window_name} #{pane_current_command} #{pane_current_path}'
```

Identify the pane actually running Codex. In my session it was `%14`; substitute your own ID below. Run this from a sibling shell inside the same tmux server:

```sh
agent_pane='%14'
TMUX_PANE="$agent_pane" workmux register-agent
TMUX_PANE="$agent_pane" workmux set-window-status working
workmux status
```

This manually marks the agent working. It repairs registration for that session; automatic updates need the hooks below.

The integration in my setup maps agent events to these commands:

| Event                   | Command                             |
| ----------------------- | ----------------------------------- |
| Session start or resume | `workmux register-agent`            |
| Prompt or tool activity | `workmux set-window-status working` |
| Permission request      | `workmux set-window-status waiting` |
| Turn finished           | `workmux set-window-status done`    |

These commands need to target the agent's tmux pane. Running registration from an unrelated shell pane can target the wrong place. My hooks use an absolute executable path and skip execution outside tmux:

```sh
if [ -n "${TMUX:-}" ] && [ -n "${TMUX_PANE:-}" ]; then
  /opt/homebrew/bin/workmux set-window-status working
fi
```

That path is for Homebrew on my Apple Silicon Mac. Check your own installation before copying it.

Find the executable path and inspect the hooks file:

```sh
command -v workmux
codex_config_dir="${CODEX_HOME:-$HOME/.codex}"
mkdir -p "$codex_config_dir"
if [ -f "$codex_config_dir/hooks.json" ]; then
  cp -p "$codex_config_dir/hooks.json" "$codex_config_dir/hooks.json.backup-$(date +%Y%m%d-%H%M%S)"
fi
nano "$codex_config_dir/hooks.json"
```

For a fresh file, paste this complete JSON. For an existing file, merge these event entries into its `hooks` object, keeping unrelated hooks. Keep one workmux action per event; remove duplicate bare workmux actions only when the equivalent guarded action is present. Replace `/opt/homebrew/bin/workmux` if the earlier command returned a different path:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "startup|resume|clear",
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux register-agent; fi"
          }
        ]
      }
    ],
    "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux set-window-status working; fi"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux set-window-status working; fi"
          }
        ]
      }
    ],
    "SubagentStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux set-window-status working; fi"
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux set-window-status working; fi"
          }
        ]
      }
    ],
    "PermissionRequest": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux set-window-status waiting; fi"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "if [ -n \"${TMUX:-}\" ] && [ -n \"${TMUX_PANE:-}\" ]; then /opt/homebrew/bin/workmux set-window-status done; fi"
          }
        ]
      }
    ]
  }
}
```

Save and exit, validate the JSON, then open the Codex configuration:

```sh
jq empty "$codex_config_dir/hooks.json"
nano "$codex_config_dir/config.toml"
```

Under the existing `[features]` table, set `hooks = true`. If that table is absent, add:

```toml
[features]
hooks = true
```

Save and exit. Restart Codex in the intended tmux pane with `codex resume`, then enter this **inside Codex**, rather than in the shell:

```text
/hooks
```

Review and enable the intended workmux hooks. After review, restart and resume once more to test the SessionStart registration. Submit a prompt and inspect `workmux status` from a sibling shell while the agent works and after it finishes. A permission request provides a chance to verify waiting status. Waiting/done indicators can clear when the window receives focus, so check them from another window. These are lifecycle checks to perform, not a claim that the complete sequence has already passed on my machine.

Codex also requires review of new or changed hook definitions. In the CLI, `/hooks` lets me inspect and trust them. Editing the JSON alone doesn't complete the integration; [the official hook documentation](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks) explains the review step.

I wanted the sidebar on the left. Reopening it with an explicit position moved it there:

```sh
workmux sidebar off
workmux sidebar --position left on
```

To save the position for future sidebar launches, run:

```sh
mkdir -p ~/.config/workmux
if [ -f ~/.config/workmux/config.yaml ]; then
  cp -p ~/.config/workmux/config.yaml ~/.config/workmux/config.yaml.backup-$(date +%Y%m%d-%H%M%S)
fi
nano ~/.config/workmux/config.yaml
```

Merge this into the existing YAML; if `sidebar` already exists, update its `position` field rather than creating another key:

```yaml
sidebar:
  position: left
```

Save and exit. This sets placement; run `workmux sidebar on` inside tmux to show it. Keep other workmux settings, including your agent command.

The left sidebar and a live working entry are verified in my session. Automatic registration on a fresh launch and the full working/waiting/done lifecycle still need an end-to-end check.

## Agent usage belongs in the agent footer

The sidebar tells me what an agent is doing. For model, context, and rate-limit information, I use the agent's own footer.

### Codex footer

Enter this **inside Codex CLI** to select and order the footer items:

```text
/statusline
```

For the version used here, the corresponding file settings go in the same `[tui]` table as the copy setting. To edit them from a shell:

```sh
codex_config_dir="${CODEX_HOME:-$HOME/.codex}"
cp -p "$codex_config_dir/config.toml" "$codex_config_dir/config.toml.backup-$(date +%Y%m%d-%H%M%S)"
nano "$codex_config_dir/config.toml"
```

Merge these fields, preserving other entries in the table:

```toml
[tui]
copy_on_select = "always"
status_line = ["model-with-reasoning", "context-used", "five-hour-limit", "weekly-limit"]
```

Save and exit, then restart Codex with `codex resume`. If your installed version doesn't support these identifiers, use its `/statusline` picker to choose the available equivalents.

### Claude Code footer

Claude Code has a separate `statusLine` configuration and a script that reads JSON from standard input. Create and open the script:

```sh
mkdir -p ~/.claude
if [ -f ~/.claude/statusline.sh ]; then
  cp -p ~/.claude/statusline.sh ~/.claude/statusline.sh.backup-$(date +%Y%m%d-%H%M%S)
fi
nano ~/.claude/statusline.sh
```

Paste the following script, save, and exit:

```bash
#!/bin/bash
jq -r '
  def pct: if type == "number" then "\(floor)%" else "n/a" end;
  [(.model.display_name // "Claude"),
   ("ctx " + (.context_window.used_percentage | pct)),
   ("5h " + (.rate_limits.five_hour.used_percentage | pct)),
   ("7d " + (.rate_limits.seven_day.used_percentage | pct))]
  | join(" | ")
'
```

Make it executable, check syntax, and open Claude's settings:

```sh
chmod +x ~/.claude/statusline.sh
bash -n ~/.claude/statusline.sh
if [ -f ~/.claude/settings.json ]; then
  cp -p ~/.claude/settings.json ~/.claude/settings.json.backup-$(date +%Y%m%d-%H%M%S)
fi
nano ~/.claude/settings.json
```

For a new settings file, paste this JSON. Otherwise, add the `statusLine` property to the existing object, preserving its other properties:

```json
{
  "statusLine": {
    "type": "command",
    "command": "~/.claude/statusline.sh"
  }
}
```

Save and exit, then validate the settings and test the script with missing and sample values:

```sh
jq empty ~/.claude/settings.json
printf '%s\n' '{}' | ~/.claude/statusline.sh
printf '%s\n' '{"model":{"display_name":"Opus"},"context_window":{"used_percentage":24},"rate_limits":{"five_hour":{"used_percentage":0},"seven_day":{"used_percentage":32}}}' |
  ~/.claude/statusline.sh
```

Expected output:

```text
Claude | ctx n/a | 5h n/a | 7d n/a
Opus | ctx 24% | 5h 0% | 7d 32%
```

Check the actual footer after a response in Claude Code. I keep missing percentages as `n/a`; unavailable usage isn't zero usage. The script reports **used** percentages, while an agent's built-in footer may show **remaining**. Check the labels before comparing them.

## What I'm leaving open

After following the steps, run these checks from a shell inside tmux:

```sh
zsh -n ~/.zshrc
zsh -n ~/.p10k.zsh
/Applications/Ghostty.app/Contents/MacOS/ghostty +validate-config
tmux show-options -g mouse
tmux show-options -s set-clipboard
workmux status
jq empty "${CODEX_HOME:-$HOME/.codex}/hooks.json"
jq empty ~/.claude/settings.json
bash -n ~/.claude/statusline.sh
```

The syntax checks should exit without errors; tmux should report `mouse on` and `set-clipboard on`. With Codex running and its hooks reviewed, workmux should list the intended agent. Confirm visually that the sidebar is on the left, the prompt has actual icons and colors, a selected word pastes correctly, and the footer appears in each agent. Watch working/waiting/done from a sibling window across a real turn; a manually set working status alone doesn't verify the hooks.

I haven't settled on a tmux prefix. I dislike both Ctrl+B and Ctrl+A, and pane navigation can wait until I decide between Neovim and Helix. The final tmux status-bar layout is open too.

For now, the useful parts are in place: the prompt is readable, copying works in the sessions I've tested, and the sidebar shows the current task. The next check is whether the agent hooks keep that status accurate across a fresh launch and a complete turn.

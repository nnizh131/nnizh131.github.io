---
title: "My macOS terminal setup: Ghostty, tmux, and workmux"
author: Nika Nizharadze
pubDatetime: 2026-10-18T08:00:00Z
modDatetime: 2026-10-19T08:00:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "My compact macOS terminal setup: Ghostty, Starship, eza, tmux, and workmux, with working clipboard support."
---

I use Ghostty, Zsh, Starship, eza, tmux, and workmux. Ghostty handles the terminal, Starship builds the prompt, and eza adds icons and Git details to directory listings. tmux keeps sessions running when I detach, while workmux keeps task checkouts and agent status together.

The terminal uses Rosé Pine Dawn and JetBrains Mono. The prompt uses a customized Gruvbox Rainbow preset, with the directory and Git state on the left and language tools and time on the right.

## Install

With Homebrew already installed:

```sh
brew install --cask ghostty
brew install starship eza tmux jq
brew install raine/workmux/workmux
```

Ghostty bundles JetBrains Mono and Nerd Font symbols. For the same font in other applications, optionally install `font-jetbrains-mono-nerd-font` through Homebrew.

Back up existing configuration files before editing. Merge the snippets below into them, keeping unrelated settings and updating existing keys instead of adding duplicates.

## Ghostty

Open the configuration through Ghostty’s settings menu and use:

```ini
font-family = JetBrains Mono
font-size = 14
theme = Rose Pine Dawn
window-padding-x = 10
window-padding-y = 8
macos-option-as-alt = true
mouse-shift-capture = never
copy-on-select = clipboard
```

Reload through the menu or with **Cmd+Shift+,**. Hold **Shift** while dragging to select terminal text when an application captures the mouse.

In Ghostty 1.3.1, clipboard writes, trailing-space trimming when copying, and the macOS **Cmd+C** and **Cmd+V** bindings already have suitable defaults. Automatic selection copying also defaults to `true`; `copy-on-select = clipboard` explicitly copies to the system clipboard. Trimming applies to Ghostty’s own copying, not application clipboard writes through OSC 52.

Shift selection already works with the default `mouse-shift-capture = false`; `never` prevents applications from overriding that behavior. `macos-option-as-alt = true` enables terminal Alt shortcuts but changes Option-based character input. Use `left` instead if you want to keep right Option available for characters such as € or ø, depending on your keyboard layout. See the [Ghostty option reference](https://ghostty.org/docs/config/reference).

## Starship and eza

Add these lines once to `~/.zshrc`, replacing the old prompt initialization:

```zsh
eval "$(starship init zsh)"

if command -v eza >/dev/null 2>&1; then
  alias ls='eza --icons=auto --group-directories-first'
  alias ll='eza --icons=auto --group-directories-first --long --git'
  alias la='eza --icons=auto --group-directories-first --all'
fi

alias mux='tmux new-session -A -s work'
alias wm='workmux'
```

`ls` shows icons with folders first, `ll` adds details and Git status, and `la` includes hidden files. `--icons=auto` shows icons in terminal output and omits them when piping or redirecting output; see the [eza manual](https://github.com/eza-community/eza/blob/main/man/eza.1.md).

Start with the Gruvbox Rainbow preset. Back up an existing Starship config first, since the preset command replaces it:

```sh
mkdir -p ~/.config
if [ -f ~/.config/starship.toml ]; then
  cp -p ~/.config/starship.toml ~/.config/starship.toml.backup-$(date +%Y%m%d-%H%M%S)
fi
starship preset gruvbox-rainbow -o ~/.config/starship.toml
```

In `~/.config/starship.toml`, replace the top-level `add_newline` and `format` values, and add or update `right_format` before the first table heading. These are my current layout settings:

```toml
add_newline = false
format = '[](color_orange)$os$username[](bg:color_yellow fg:color_orange)$directory[](fg:color_yellow bg:color_aqua)$git_branch$git_status[](fg:color_aqua) $character'
right_format = '[](color_blue)$c$cpp$rust$golang$nodejs$bun$php$java$kotlin$haskell$python[](fg:color_blue bg:color_bg3)$docker_context$conda$pixi[](fg:color_bg3 bg:color_bg1)$time[](fg:color_bg1)'
```

Keep the preset’s palette definitions: these formats reference its named colors. To simplify the prompt, remove unwanted `$module` references from `format` or `right_format`. Deleting a module’s configuration table alone can leave it active with default styling. With these explicit formats, the preset’s unused Jujutsu and line-break tables can be removed as optional housekeeping. See the [Starship configuration reference](https://starship.rs/config/).

Check and reload the shell:

```sh
zsh -n ~/.zshrc
source ~/.zshrc
```

## tmux and copying

Use these settings in `~/.tmux.conf`. I’m keeping the default prefix for now.

```text
set -g mouse on
set -g history-limit 100000
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on
set -g focus-events on
set -g default-terminal 'tmux-256color'
set -as terminal-features ',xterm-ghostty:RGB:clipboard'
set -s set-clipboard on
setw -g mode-keys vi

bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
```

The mouse-release bindings cover both Vi and Emacs copy mode. With `mode-keys vi`, the Vi binding handles normal copy mode; the second binding also covers sessions using Emacs copy mode. Keeping both fixed copying in my earlier session, but both are only needed when using both tables. These `pbcopy` bindings write directly to the local macOS clipboard.

In tmux 3.8, `escape-time` already defaults to 10 milliseconds, so it needs no override. Its default `xterm*` terminal features also include clipboard support, matching `xterm-ghostty`. The explicit `RGB:clipboard` declaration keeps these capabilities stated in the config; it may be redundant when tmux detects them automatically. Check `tmux display-message -p '#{client_termname} #{client_termfeatures}'` from an attached client before removing it. See the [tmux 3.8 defaults](https://github.com/tmux/tmux/blob/3.8/options-table.c).

`set-clipboard on` enables a separate OSC 52 path, allowing applications to create tmux buffers and forward clipboard writes to the outer terminal. Successful copying through `pbcopy` does not verify that path. Local `pbcopy` bindings also do not copy to your Mac when tmux runs on a remote host; remote clipboard integration needs OSC 52 support through the terminal chain.

From outside tmux, create or attach to the session:

```sh
tmux new-session -A -s work
```

Inside tmux, apply changes:

```sh
tmux source-file ~/.tmux.conf
```

New terminal and history settings may require new panes. Existing tasks can keep running; there’s no need to kill the server. Select a harmless word and run `pbpaste` to check copying.

## workmux

Inside tmux:

```sh
workmux list
workmux status
workmux dashboard
workmux sidebar --position left on
```

Press `q` to leave the dashboard. Use `workmux add my-feature` to create a task checkout, or `workmux open my-feature` to reopen one.

To keep the sidebar on the left, merge this into `~/.config/workmux/config.yaml`:

```yaml
sidebar:
  position: left
```

If the sidebar says **No agents running**, check registration first. From a sibling shell, find the pane actually running the agent:

```sh
tmux list-panes -a -F '#{pane_id} #{window_name} #{pane_current_command} #{pane_current_path}'
```

Substitute that pane ID below:

```sh
agent_pane='%14'
TMUX_PANE="$agent_pane" workmux register-agent
TMUX_PANE="$agent_pane" workmux set-window-status working
workmux status
```

That repairs registration and manually sets the status for the session. Automatic updates depend on agent hooks:

| Agent event             | workmux action                      |
| ----------------------- | ----------------------------------- |
| Session start or resume | `workmux register-agent`            |
| Prompt or tool activity | `workmux set-window-status working` |
| Permission request      | `workmux set-window-status waiting` |
| Turn finished           | `workmux set-window-status done`    |

Hooks must target the agent’s pane and skip execution outside tmux. For Codex, review and trust changed hooks through `/hooks`; see the [hook review instructions](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks). The sidebar and manual working status are verified in my session; fresh-launch registration and the full status lifecycle still need an end-to-end check.

## Agent footer and clipboard

In my Codex CLI 0.163.0 setup, these settings share the existing `[tui]` table in `${CODEX_HOME:-~/.codex}/config.toml`:

```toml
[tui]
copy_on_select = "always"
status_line = ["model-with-reasoning", "context-used", "five-hour-limit", "weekly-limit"]
```

Restart Codex and use `codex resume` to return to the conversation. The `/statusline` picker lets you choose the footer items supported by your version. Copying works in my direct Codex and tmux sessions.

Claude Code has its own footer and clipboard settings. Its clipboard path still needs checking in my session.

I’m leaving the tmux prefix, editor navigation, and final status-bar layout open for now.

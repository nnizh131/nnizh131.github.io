---
title: "Terminal Setup: Ghostty, tmux, and workmux"
author: Nika Nizharadze
pubDatetime: 2026-10-18T08:00:00Z
modDatetime: 2026-10-19T08:00:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "A practical macOS terminal setup with Ghostty, Zsh, Starship, eza, tmux, and workmux, including clipboard integration and agent tracking. Part 1 of a three-part series."
---

![Terminal demo showing Codex startup, a prompt and answer, the workmux sidebar, and dashboard tabs.](/terminal-setup/terminal-setup-full.gif)

A useful terminal setup needs readable output, persistent sessions, and a way to keep parallel tasks separate. I use Ghostty as the GPU-accelerated terminal, Zsh with Starship and eza for the shell experience, tmux for persistent sessions, and workmux to connect Git worktrees with agent workflows.

This is a three-part series:

1. [Terminal Setup: Ghostty, tmux, and workmux](/posts/macos-terminal-setup/)
2. [Terminal Shortcuts: Keybindings for Ghostty and tmux](/posts/macos-terminal-shortcuts/)
3. [Terminal Workflow: tmux, workmux, and Codex](/posts/macos-terminal-workflow/)

## 1. Installation

```sh
brew install --cask ghostty
brew install starship eza tmux jq
brew install raine/workmux/workmux
```

Ghostty bundles JetBrains Mono and Nerd Font symbols. For the same font in an IDE or other macOS applications, optionally install:

```sh
brew install --cask font-jetbrains-mono-nerd-font
```

## 2. Ghostty configuration

Open Ghostty’s configuration through its settings menu or **Cmd+,** and set the font, theme, spacing, and selection behavior:

```ini
font-family = "JetBrains Mono"
font-size = 14
theme = "Rose Pine Dawn"
window-padding-x = 10
window-padding-y = 8
macos-option-as-alt = true
mouse-shift-capture = never
copy-on-select = clipboard
```

Reload through the menu or with **Cmd+Shift+,**. Hold **Shift** while dragging to select terminal text when an application captures the mouse.

## 3. Zsh, Starship, and eza

### Step A: Update ~/.zshrc

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
alias codex='codex --no-daemon'
```

`ls` shows icons with folders first, `ll` adds details and Git status, and `la` includes hidden files. `mux` creates or attaches to the tmux session named `work`, and `wm` is a shorter name for `workmux`. The Codex alias uses the launch mode that fixed automatic agent registration in my setup; the workmux section below explains why. `--icons=auto` shows icons in terminal output and omits them when piping or redirecting output; see the [eza manual](https://github.com/eza-community/eza/blob/main/man/eza.1.md).

### Step B: Configure the Starship prompt

Start with the Gruvbox Rainbow preset:

```sh
mkdir -p ~/.config
starship preset gruvbox-rainbow -o ~/.config/starship.toml
```

In `~/.config/starship.toml`, replace the top-level `add_newline` and `format` values, and add or update `right_format` before the first table heading. These are my current layout settings:

```toml
add_newline = false
format = '[](color_orange)$os$username[](bg:color_yellow fg:color_orange)$directory[](fg:color_yellow bg:color_aqua)$git_branch$git_status[](fg:color_aqua) $character'
right_format = '[](color_blue)$c$cpp$rust$golang$nodejs$bun$php$java$kotlin$haskell$python[](fg:color_blue bg:color_bg3)$docker_context$conda$pixi[](fg:color_bg3 bg:color_bg1)$time[](fg:color_bg1)'
```

Keep the preset’s palette definitions: these formats reference its named colors. To simplify the prompt, remove unwanted `$module` references from `format` or `right_format`. Deleting a module’s configuration table alone can leave it active with default styling. With these explicit formats, the preset’s unused Jujutsu and line-break tables can be removed as optional housekeeping. See the [Starship configuration reference](https://starship.rs/config/).

Replace the preset’s existing `[git_status]` section with these settings to show counts beside each Git status symbol:

```toml
[git_status]
style = "bg:color_aqua"
format = '[[($all_status$ahead_behind )](fg:color_fg0 bg:color_aqua)]($style)'
staged = '+$count '
modified = '!$count '
untracked = '?$count '
deleted = '-$count '
renamed = '»$count '
conflicted = '~$count '
stashed = '*$count '
ahead = '⇡$count '
behind = '⇣$count '
diverged = '⇕⇡$ahead_count⇣$behind_count '
```

For example, `+2 !1 ?1` means two staged files, one modified file, and one untracked file. `-1` indicates one deleted file. The stash symbol counts stashes, while the arrows count commits ahead of or behind the upstream branch. Statuses with a zero count are hidden. Starship reads configuration changes on the next prompt.

Check and reload the shell:

```sh
zsh -n ~/.zshrc
source ~/.zshrc
```

## 4. tmux configuration and macOS clipboard

tmux keeps its own copy buffers. To copy selections to the macOS system clipboard, the bindings below send text directly to `/usr/bin/pbcopy`. Use these settings in `~/.tmux.conf`; they enable mouse support and Vi copy mode while keeping the default **Ctrl+B** prefix.

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

Mouse support lets me click panes to focus them, drag pane borders to resize them, and select text. `base-index` starts window numbering at 1, `pane-base-index` does the same for panes, and `renumber-windows` closes numbering gaps after a window is removed.

## 5. workmux setup

workmux connects isolated Git worktrees with tmux windows, so each task can have its own checkout, agent, and shell. Run these commands from a shell inside tmux:

| Command                              | Purpose                                   |
| ------------------------------------ | ----------------------------------------- |
| `workmux list`                       | List task worktrees.                      |
| `workmux status`                     | Check tracked agent status.               |
| `workmux dashboard`                  | Open the management dashboard.            |
| `workmux add my-feature`             | Create a task worktree and its workspace. |
| `workmux open my-feature`            | Return to an existing task.               |
| `workmux sidebar --position left on` | Enable the sidebar on the left.           |

Press `q` to leave the dashboard. The dashboard and sidebar are separate views: opening the dashboard does not enable the sidebar. Use `workmux add my-feature` to create a task checkout, or `workmux open my-feature` to reopen one.

### Customizing the sidebar

To keep the sidebar on the left and show agent identity, merge this into `~/.config/workmux/config.yaml`:

```yaml
nerdfont: true
sidebar:
  position: left
  templates:
    compact: "{status_icon} {agent_icon} {primary} {pane_suffix} {fill} {elapsed}"
    tiles:
      - "{agent_icon} {primary} {pane_suffix} {fill} {elapsed}"
      - "{secondary} {fill} {git_stats}"
      - "{pane_title} {fill} {pr_number} {pr_checks}"
    grouped:
      tiles:
        - "{agent_icon} {primary} {pane_suffix} {fill} {pr_number} {pr_checks} {elapsed}"
        - "{pane_title} {fill} {git_stats}"
```

The sidebar keeps agent identity, elapsed time, and Git diff information beside the task panes. `{agent_icon}` shows identity, such as **CX** for Codex. `nerdfont: true` changes status symbols; it does not replace the CX label with a logo. Unknown agent identities render an empty icon. My Codex pane appeared as `node`, which workmux did not identify as Codex; correcting its current registration and adding the template field made CX visible. That identity correction was specific to the running session.

The timer is **elapsed time since the last status change**, including time spent done or idle. It keeps counting without new messages and does not indicate ongoing processing or token use. Remove `{elapsed}` from the templates to hide it. The done indicator clears when you focus the agent window, so a missing check mark does not mean the agent restarted. See [sidebar customization](https://workmux.raine.dev/guide/sidebar/customization/).

### Hooks and missing agents

Install status tracking with:

```sh
workmux setup --hooks
```

A **configured** result confirms installation. For Codex, also check `/hooks` for enabled hooks and review any changed definitions; see the [hook review instructions](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks).

| Agent event             | workmux action                      |
| ----------------------- | ----------------------------------- |
| Session start or resume | `workmux register-agent`            |
| Prompt or tool activity | `workmux set-window-status working` |
| Permission request      | `workmux set-window-status waiting` |
| Turn finished           | `workmux set-window-status done`    |

In my Ghostty, tmux 3.8, workmux 0.1.276, and Codex CLI 0.163.0 session, the hooks were installed, enabled, and firing, but the sidebar still showed **No agents running**. The workmux log showed `tmux_pane=None`: the hook processes had lost their tmux pane context. Workmux could not associate those events with the running agent. Codex was using its shared background server, so the hook processes were outside the agent’s tmux process tree. Reinstalling or trusting the same hooks again did not address that missing association.

Starting Codex without the shared server fixed automatic registration in my setup. Run this from the tmux pane where you want the agent:

```sh
codex --no-daemon
```

The agent then appeared automatically in the sidebar. To use this launch mode whenever I type `codex` in Zsh, I added this to `~/.zshrc`:

```zsh
alias codex='codex --no-daemon'
```

Open a new shell or run `source ~/.zshrc` to load it. This alias affects interactive shell commands; scripts and workmux configurations that launch Codex directly should use `codex --no-daemon` explicitly.

To check the setup, start a new Codex session inside tmux and confirm it appears in the sidebar without manual registration. Send a prompt and check that its status moves from working to done. Automatic registration and these transitions are verified in my setup. The permission-request waiting state and persistent identity detection still need an end-to-end check. See the [upstream status-tracking guide](https://workmux.raine.dev/guide/status-tracking/).

## 6. Agent footer and clipboard

In my Codex CLI 0.163.0 setup, these settings share the existing `[tui]` table in `${CODEX_HOME:-~/.codex}/config.toml`:

```toml
[tui]
copy_on_select = "always"
status_line = ["model-with-reasoning", "context-used", "five-hour-limit", "weekly-limit"]
```

Restart Codex and use `codex resume` to return to the conversation. The `/statusline` picker lets you choose the footer items supported by your version. Copying works in my direct Codex and tmux sessions.

Claude Code has its own footer and clipboard settings. Its clipboard path still needs checking in my session.

I’m leaving the tmux prefix, editor navigation, and final status-bar layout open for now.

Continue with [Part 2: Shortcuts](/posts/macos-terminal-shortcuts/) for the keybindings I use with this setup, or [Part 3: Daily workflow](/posts/macos-terminal-workflow/) for the task-by-task flow.

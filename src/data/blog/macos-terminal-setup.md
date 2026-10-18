---
title: "My macOS terminal setup: Ghostty, tmux, and workmux"
author: Nika Nizharadze
pubDatetime: 2026-10-18T08:00:00Z
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
brew install --cask font-jetbrains-mono
```

I ran the installs sequentially and backed up the existing configuration files before editing them. I kept unrelated settings rather than replacing whole files with a new set of dotfiles.

## A light terminal and a familiar prompt

My active Ghostty config is under `~/Library/Application Support/com.mitchellh.ghostty/config`. Check the active path through Ghostty's settings before editing; another installation may use `~/.config/ghostty/config`.

These are the main appearance settings:

```ini
font-family = JetBrains Mono
font-size = 14
theme = Rose Pine Dawn
window-padding-x = 10
window-padding-y = 8
macos-option-as-alt = true
```

Size 14 is what I'm trying. Dawn's background is a very pale cream, `#faf4ed`. Ghostty supplies the symbol fallback for the icons in my prompt, so I didn't install a separate patched font.

I kept the existing single-line, segmented Powerlevel10k layout. It already showed the things I wanted: macOS, current directory, Git branch, and command status. I changed its fixed colors to fit Dawn:

| Prompt element                | Color     |
| ----------------------------- | --------- |
| Segment background            | `#f2e9e1` |
| Main text                     | `#575279` |
| Directory and clean Git state | `#286983` |
| OS icon and untracked count   | `#907aa9` |
| Modifications and errors      | `#b4637a` |

The custom Git formatter had its own color escapes. Those needed changing too. One broken attempt printed a color escape as literal text; the valid Zsh form is `%F{#797593}`, followed by `%f` to reset.

Editing the prompt arrays also caught me out. A replacement stopped at a parenthesis inside a comment and left old entries behind. Now I check the whole array and run:

```sh
zsh -n ~/.zshrc
zsh -n ~/.p10k.zsh
```

After validation, sourcing `~/.p10k.zsh` in the interactive shell reloads the prompt. Ghostty settings reload with `Cmd+Shift+,`.

## Getting copying to work

There were three places to check: Ghostty, tmux, and the agent interface.

For native Ghostty selection, I configured:

```ini
mouse-shift-capture = never
clipboard-write = allow
clipboard-trim-trailing-spaces = true
copy-on-select = clipboard
keybind = super+c=copy_to_clipboard
keybind = super+v=paste_from_clipboard
```

Holding Shift while dragging lets me make a native terminal selection when an application captures the mouse. The exact options are documented in [Ghostty's configuration reference](https://ghostty.org/docs/config/reference).

Inside tmux, I needed mouse support and clipboard bindings for both copy-mode tables:

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

The earlier configuration only covered Vi copy mode, while the running server was using Emacs copy mode. Covering both fixed that gap.

`set-clipboard on` also allows applications inside tmux to write through OSC 52. The `external` setting blocks application clipboard requests. That distinction mattered for Codex; see the [tmux clipboard guide](https://github.com/tmux/tmux/wiki/Clipboard). The `pbcopy` bindings above target a local macOS session.

For Codex CLI 0.163.0, I added this to the existing table in `~/.codex/config.toml`:

```toml
[tui]
copy_on_select = "always"
```

I restarted Codex and tested selecting a response and pasting into another window. Copying now works in my direct Codex session and inside tmux. I still need to test the separate Claude Code clipboard path.

## The sidebar needs a registered agent

The workmux sidebar initially said **No agents running**, even though Codex was open. The pane hadn't been registered with workmux. Registering the actual Codex pane made the entry appear, with its working status.

That gives me a useful debugging order: check `workmux status` before debugging the sidebar itself. An empty registry explains an empty sidebar.

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

Codex also requires review of new or changed hook definitions. In the CLI, `/hooks` lets me inspect and trust them. Editing the JSON alone doesn't complete the integration; [the official hook documentation](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks) explains the review step.

I wanted the sidebar on the left. Reopening it with an explicit position moved it there:

```sh
workmux sidebar off
workmux sidebar --position left on
```

The left sidebar and a live working entry are verified in my session. Automatic registration on a fresh launch and the full working/waiting/done lifecycle still need an end-to-end check.

## Agent usage belongs in the agent footer

The sidebar tells me what an agent is doing. For model, context, and rate-limit information, I use the agent's own footer.

In Codex CLI, `/statusline` lets me select those items. Claude Code has a separate `statusLine` configuration and a script that reads JSON from standard input. I keep missing percentages as `n/a`; unavailable usage isn't zero usage. I also check whether a label means used or remaining before comparing numbers.

## What I'm leaving open

I haven't settled on a tmux prefix. I dislike both Ctrl+B and Ctrl+A, and pane navigation can wait until I decide between Neovim and Helix. The final tmux status-bar layout is open too.

For now, the useful parts are in place: the prompt is readable, copying works in the sessions I've tested, and the sidebar shows the current task. The next check is whether the agent hooks keep that status accurate across a fresh launch and a complete turn.

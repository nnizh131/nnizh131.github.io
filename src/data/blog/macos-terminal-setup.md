---
title: "Terminal Setup: Ghostty, tmux, and workmux"
author: Nika Nizharadze
pubDatetime: 2026-10-18T08:00:00Z
modDatetime: 2026-07-25T08:00:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "My macOS terminal setup: Ghostty, a colorful Rosé Pine Dawn prompt, tmux, workmux, Codex, and Claude Code, with downloadable configs. Part 1 of three."
---

![Rosé Pine Dawn terminal demo: colorful Starship prompt, Codex completing a task, workmux sidebar, and dashboard diff review.](/terminal-setup/terminal-setup-dawn-colorful.gif)

[Watch the video with playback controls](/terminal-setup/terminal-setup-dawn-colorful.mp4). The demo renders a real tmux, workmux, and Codex session with the Dawn palette and updated prompt colors.

This is a three-part series:

1. [Terminal Setup: Ghostty, tmux, and workmux](/posts/macos-terminal-setup/)
2. [Terminal Shortcuts: Keybindings for Ghostty and tmux](/posts/macos-terminal-shortcuts/)
3. [Terminal Workflow: tmux, workmux, and Codex](/posts/macos-terminal-workflow/)

I use Ghostty for the terminal window, Zsh for the shell, Starship for the prompt, eza for file listings, tmux for persistent sessions, and workmux for task worktrees. Codex or Claude Code runs inside a task pane.

Each layer controls a different part of the appearance. Ghostty sets the font, size, and terminal colors. Starship styles only the shell prompt; eza supplies file icons. Codex and workmux have their own themes. A theme repository’s screenshot may also include a system-info tool—installing its colors does not reproduce the entire screenshot.

## 1. Install the tools

With [Homebrew](https://brew.sh/) installed:

```sh
brew install --cask ghostty font-jetbrains-mono-nerd-font codex
brew install starship eza tmux jq
brew install raine/workmux/workmux
```

Open Codex once and sign in when prompted. Back up existing configuration files before merging the examples below.

## 2. Ghostty: font and light theme

Open Ghostty’s settings with **Cmd+,** and edit the active configuration:

```ini
font-family = "JetBrainsMono Nerd Font Mono"
font-size = 14
theme = "Rose Pine Dawn"
window-padding-x = 10
window-padding-y = 8
macos-option-as-alt = true
mouse-shift-capture = never
copy-on-select = clipboard
```

Reload with **Cmd+Shift+,**. Dawn is the light Rosé Pine variant, with a cream background (`#faf4ed`). The Nerd Font provides the prompt and file-list symbols; changing the theme does not change their size. See the [Ghostty settings reference](https://ghostty.org/docs/config/reference) and [Rosé Pine theme](https://github.com/rose-pine/ghostty).

## 3. Zsh: colorful Starship prompt and eza

Add these lines once to `~/.zshrc`, replacing any previous prompt initialization:

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

`ll` adds details and Git status; `la` includes hidden files. `mux` creates or attaches to the `work` session. The Codex alias preserves tmux context for agent tracking in my setup, as explained below.

Save the [complete Starship config](/terminal-setup/starship-rose-pine-dawn.toml) as `~/.config/starship.toml`, or merge it with your existing config. It keeps the [Gruvbox Rainbow layout](https://starship.rs/presets/gruvbox-rainbow), recolored with Rosé Pine: teal for the system segment, rose for the directory, and purple for Git, all with light text. Languages and the clock sit on the right.

These are the key palette values in that file:

```toml
[palettes.rose_pine_dawn]
color_fg0 = "#fffaf3"
color_orange = "#286983" # system
color_yellow = "#b4637a" # directory
color_aqua = "#907aa9"   # Git
color_blue = "#56949f"   # languages
color_bg3 = "#b4637a"    # environments
color_bg1 = "#907aa9"    # clock
```

The names match the original preset; their values produce the colorful Dawn prompt shown above. Changing Ghostty’s theme alone will not change these explicit colors. Use the full download for the remaining palette values and module settings.

Git counts are compact: `+2 !1 ?1` means two staged files, one modified file, and one untracked file. `-` counts deletions, `»` renames, `~` conflicts, `*` stashes, and `⇡`/`⇣` commits ahead/behind upstream. Zero counts stay hidden. Remove unwanted `$module` references from `format` or `right_format` to simplify it; see the [Starship reference](https://starship.rs/config/).

```sh
zsh -n ~/.zshrc
source ~/.zshrc
```

Starship reads its configuration on the next prompt.

## 4. tmux: sessions, colors, and clipboard

Merge this into `~/.tmux.conf`. It keeps the default **Ctrl+B** prefix and sends copied selections to the macOS clipboard:

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

set -g window-style 'bg=#faf4ed,fg=#575279'
set -g window-active-style 'bg=#faf4ed,fg=#575279'
set -g status-style 'bg=#f2e9e1,fg=#575279'
set -g pane-border-style 'fg=#cecacd'
set -g pane-active-border-style 'fg=#907aa9'

bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
bind -T copy-mode MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel '/usr/bin/pbcopy'
```

The explicit Dawn pane background also lets applications detect a light terminal correctly. Mouse support enables pane focus, border resizing, and selection. Windows and panes start at 1.

For an existing session, run `tmux source-file ~/.tmux.conf`; new panes pick up the terminal setting. Hold **Shift** while dragging for Ghostty-native selection, and paste with **Cmd+V**. [Part 2](/posts/macos-terminal-shortcuts/) covers keyboard copying.

## 5. workmux: task panes, sidebar, and dashboard

Merge the [workmux Dawn config](/terminal-setup/workmux-rose-pine-dawn.yaml) into `~/.config/workmux/config.yaml`. It includes the sidebar templates and Dawn colors, with a focused upper pane for starting Codex and a seven-line shell below. Project-level `.workmux.yaml` settings override global defaults. See the [configuration reference](https://workmux.raine.dev/guide/configuration/).

From a shell inside tmux:

| Command                              | Purpose                              |
| ------------------------------------ | ------------------------------------ |
| `workmux add my-feature`             | Create a task worktree and window.   |
| `workmux open my-feature`            | Return to an existing task.          |
| `workmux list`                       | List worktrees.                      |
| `workmux status`                     | Check tracked agents.                |
| `workmux sidebar --position left on` | Show the left sidebar.               |
| `workmux dashboard --session`        | Inspect agents in this tmux session. |

The sidebar shows agent identity, status, elapsed time, and Git changes. **CX is workmux’s Codex label**, not a missing logo; `nerdfont: true` changes status symbols. The dashboard is a separate view: use **Tab** for Agents/Worktrees, **d** for a diff, and **q** to go back. Omit `--session` to see agents across sessions.

Install tracking hooks:

```sh
workmux setup --hooks
```

In Codex, use `/hooks` to inspect and enable the intended workmux hooks; changed definitions need review. Hooks register the session and update working, waiting, and done states. See the [official hook review instructions](https://learn.chatgpt.com/docs/hooks#review-and-trust-hooks).

If the sidebar says **No agents running**, launch `codex --no-daemon` inside the task pane. With tmux 3.8, workmux 0.1.276, and Codex 0.163.0, my shared Codex server lost the hooks’ tmux pane context (`tmux_pane=None`). This launch mode restored automatic registration; reinstalling the same hooks did not.

## 6. Codex: matching theme and footer

Install the [Rosé Pine Dawn TextMate theme](https://github.com/rose-pine/tm-theme):

```sh
mkdir -p "${CODEX_HOME:-$HOME/.codex}/themes"
curl -fL \
  https://raw.githubusercontent.com/rose-pine/tm-theme/main/dist/rose-pine-dawn.tmTheme \
  -o "${CODEX_HOME:-$HOME/.codex}/themes/rose-pine-dawn.tmTheme"
```

Run `/theme` and select Rosé Pine Dawn. It styles code blocks and diffs independently of Ghostty. Merge these settings into the existing `[tui]` table in `$CODEX_HOME/config.toml` (normally `~/.codex/config.toml`):

```toml
[tui]
theme = "rose-pine-dawn"
copy_on_select = "always"
status_line = []
```

The demo hides the optional footer. Use `/statusline` to choose model, context, or usage items for your own layout. Restart Codex after editing its config; `codex resume` returns to a conversation. See the [official theme instructions](https://learn.chatgpt.com/docs/cli-customization).

## 7. Claude Code: Dawn colors and workmux

Claude Code can use the same terminal and task layout. Install it with the [official native installer](https://code.claude.com/docs/en/setup#install-claude-code), then start it from your project and sign in:

```sh
curl -fsSL https://claude.ai/install.sh | bash
claude --version
claude
```

**Match the theme.** Run `/theme` inside Claude Code. The built-in `light-ansi` option uses terminal ANSI colors, but does not reproduce every Dawn color. For a custom palette, create `~/.claude/themes/rose-pine-dawn.json` with this light-based theme:

```json
{
  "name": "Rosé Pine Dawn",
  "base": "light",
  "overrides": {
    "claude": "#907aa9",
    "claudeShimmer": "#b4637a",
    "success": "#286983",
    "error": "#b4637a",
    "warning": "#a46b19",
    "userMessageBackground": "#f2e9e1"
  }
}
```

Run `/theme` again and select Rosé Pine Dawn (`custom:rose-pine-dawn`). The corresponding setting is `"theme": "custom:rose-pine-dawn"` in `~/.claude/settings.json`; merge that key into the existing file, keeping permissions, hooks, and other settings. The custom theme controls Claude Code’s interface accents; Ghostty and tmux still control the terminal background. These tokens are present in my installed Claude Code 2.1.295. The Iris accent on permission prompts and the startup spinner still needs a visual check.

**Keep truecolor inside tmux.** Use the `xterm-ghostty:RGB` terminal feature from section 4, then launch Claude Code with the variable already in its environment:

```sh
CLAUDE_CODE_TMUX_TRUECOLOR=1 claude
```

For future shell launches, add `export CLAUDE_CODE_TMUX_TRUECOLOR=1` to `~/.zshrc` and open a new shell. Setting it before startup also covers versions that read color capabilities before loading settings. See the [environment variable reference](https://code.claude.com/docs/en/env-vars). Keep the explicit Dawn pane colors from section 4 for this setup; `window-colorspace = display-p3` is a separate Ghostty display option, not a Claude Code theme requirement.

**Launch through workmux.** To start Claude Code automatically in the upper pane, merge this into your workmux configuration, retaining the existing theme and sidebar settings:

```yaml
agent: claude
panes:
  - command: <agent>
    focus: true
  - split: vertical
    size: 7
```

This replaces the `panes` list from section 5. New tasks start Claude Code above the seven-line shell. To choose Codex for a task instead, use `workmux add my-feature --agent codex`. See the [agent configuration reference](https://workmux.raine.dev/guide/agents/).

Install the [workmux status plugin](https://workmux.raine.dev/guide/status-tracking/#claude-code-setup) if Claude Code tracking is not already configured:

```sh
claude plugin marketplace add raine/workmux
claude plugin install workmux-status
```

Keep an existing workmux hook integration rather than installing duplicate tracking hooks. Restart Claude Code after changing its startup environment or installing the plugin; `claude --continue` resumes the most recent conversation in the current directory.

To check the result, open a new task inside tmux, confirm `/theme` selects the custom theme, and compare the message surface and accents with a Claude Code session outside tmux. Run a small task and check `workmux status` for its working and completion states. Theme selection alone does not verify every prompt or spinner color.

## Quick checks

- **Pale prompt:** use the colorful Starship palette above; Ghostty cannot override fixed prompt colors.
- **Missing color:** remove an unintended `NO_COLOR` export, check `echo "$TERM"` (`tmux-256color` inside tmux), and use `git diff --color=always` when capturing a diff.
- **Dark diff blocks on a light background:** reload the tmux Dawn pane colors, restart Codex, and ensure workmux has `theme.mode: light`.
- **Missing symbols:** select the installed Nerd Font in Ghostty. The sidebar’s CX label is expected.
- **Clipboard:** copy a harmless selection and paste it into another app to verify the complete path.

Continue with [Part 2: Shortcuts](/posts/macos-terminal-shortcuts/) or [Part 3: Daily workflow](/posts/macos-terminal-workflow/).

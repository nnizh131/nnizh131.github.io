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

New terminal and history settings may require new panes. Existing tasks can keep running; there’s no need to kill the server.

Test each local copying path separately with a harmless word:

1. **tmux keyboard:** press **Ctrl+B**, then **[** to enter copy mode. Navigate to the word, press **v**, extend the selection, and press **y**. Run `pbpaste` to confirm the selected text.
2. **tmux mouse:** drag across the word and release, then run `pbpaste`.
3. **Ghostty selection:** hold **Shift** while dragging, press **Cmd+C**, then run `pbpaste`.

Each test should print the selected text. These checks verify local copying; they do not verify the remote OSC 52 path.

## workmux

Inside tmux:

```sh
workmux list
workmux status
workmux dashboard
workmux sidebar --position left on
```

Press `q` to leave the dashboard. The dashboard and sidebar are separate views: opening the dashboard does not enable the sidebar. Use `workmux add my-feature` to create a task checkout, or `workmux open my-feature` to reopen one.

### Sidebar appearance

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

`{agent_icon}` shows identity, such as **CX** for Codex. `nerdfont: true` changes status symbols; it does not replace the CX label with a logo. Unknown agent identities render an empty icon. My Codex pane appeared as `node`, which workmux did not identify as Codex; correcting its current registration and adding the template field made CX visible. That identity correction was specific to the running session.

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

Open a new shell or run `source ~/.zshrc` to load it. This alias affects interactive shell commands; scripts and workmux configurations that launch Codex directly should use `codex --no-daemon` explicitly. The session-ID binding below is a workaround for an existing session launched through the shared server.

Find the agent pane from another shell inside the **same tmux server**:

```sh
tmux list-panes -a -F '#{pane_id} #{window_name} #{pane_current_command} #{pane_current_path}'
```

For a temporary registration, substitute the confirmed pane ID:

```sh
agent_pane='%14'
TMUX_PANE="$agent_pane" workmux register-agent
workmux status
```

This makes the agent visible, but hooks that lack pane context may still fail to update it. In my session, binding the exact Codex session ID to that pane allowed subsequent hooks to resolve it by session ID. From the same tmux server, replace both placeholders with the confirmed values:

```sh
agent_pane='%14'
agent_session='REPLACE_WITH_EXACT_CODEX_SESSION_ID'
agent_socket="$(tmux display-message -p '#{socket_path}')"
printf '{"session_id":"%s"}\n' "$agent_session" |
  env WORKMUX_STATUS_BACKEND=tmux \
      WORKMUX_STATUS_INSTANCE="$agent_socket" \
      WORKMUX_STATUS_PANE_ID="$agent_pane" \
      workmux register-agent
```

Use the Codex session ID from that conversation’s session metadata, not its title or tmux pane ID. Do not guess a pane from the working directory: multiple agents can share it. This binding repairs the current session; it does not establish a permanent fix for fresh sessions.

After binding, new prompts produced working → done transitions automatically in my session. A status update also resolved the pane with the tmux targeting variables removed. Automatic registration was subsequently verified with `codex --no-daemon`. The permission-request waiting state and persistent identity detection still need an end-to-end check. See the [upstream status-tracking guide](https://workmux.raine.dev/guide/status-tracking/).

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

## Shortcuts and workflow

I keep one tmux session for work and a separate workmux worktree for each task. Ghostty is the terminal window; tmux windows hold the tasks, and panes hold the agent and shell side by side.

### Shortcuts

The tmux prefix is **Ctrl+B**. Press it, release it, then press the next key. Copy-mode keys apply after entering copy mode.

| Shortcut                         | Action                                                    |
| -------------------------------- | --------------------------------------------------------- |
| **Ctrl+B**, then **c**           | Open a new shell window in tmux.                          |
| **Ctrl+B**, then **n** / **p**   | Switch to the next / previous window.                     |
| **Ctrl+B**, then a window number | Switch directly to that window.                           |
| **Ctrl+B**, then **o**           | Move to the next pane, including the sidebar.             |
| **Ctrl+B**, then **z**           | Zoom the current pane; repeat to restore the layout.      |
| **Ctrl+B**, then **[**           | Enter copy mode to navigate scrollback and select text.   |
| **v**, then move, then **y**     | Select and copy in Vi copy mode using the bindings above. |
| **Shift+drag**, then **Cmd+C**   | Select and copy through Ghostty.                          |
| **Cmd+V**                        | Paste from the macOS clipboard.                           |
| **Ctrl+B**, then **d**           | Detach from tmux while leaving its tasks running.         |
| **q** in the workmux dashboard   | Close the dashboard and return to the shell.              |

With tmux mouse support enabled, I can also click a pane to focus it and drag pane borders to resize them.

### Daily flow

1. **Start or return to work.** Open Ghostty, change to the project repository, and attach to the session:

   ```sh
   tmux new-session -A -s work
   ```

2. **Create a task.** From a shell inside tmux, create a worktree and its window:

   ```sh
   workmux add my-feature
   ```

   Use a descriptive task name instead of `my-feature`. To return to an existing task, run `workmux open my-feature`.

3. **Start the agent.** In the task’s agent pane, run:

   ```sh
   codex --no-daemon
   ```

   With the Zsh alias above loaded, `codex` is enough. Give it the task, then use the other shell pane for commands, tests, and reviewing changes. If the agent is already running, continue that session.

4. **Keep the sidebar visible.** Run this from a shell pane inside tmux:

   ```sh
   workmux sidebar --position left on
   ```

   The hooks register the agent and update its status. The identity label tells me which agent is running; the timer shows time since its last status change. For a larger view, run `workmux dashboard` and press **q** when finished.

5. **Review and switch tasks.** Read the agent’s changes, run the project’s checks in the shell pane, and inspect `git diff` before committing. Switch windows with the shortcuts above or use `workmux open` to return to another worktree. Copy terminal output with tmux copy mode or Ghostty’s Shift selection.

6. **Leave and resume.** Press **Ctrl+B**, then **d** to detach. Later, run `tmux new-session -A -s work` again. The session stays available while the tmux server and Mac remain running; it does not survive a reboot by itself.

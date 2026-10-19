---
title: "Terminal Workflow: tmux, workmux, and Codex"
author: Nika Nizharadze
pubDatetime: 2026-10-19T08:02:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "My daily flow for starting tmux, creating workmux tasks, running Codex, reviewing changes, and returning to work. Part 3 of a three-part series."
---

This is a three-part series:

1. [Terminal Setup: Ghostty, tmux, and workmux](/posts/macos-terminal-setup/)
2. [Terminal Shortcuts: Keybindings for Ghostty and tmux](/posts/macos-terminal-shortcuts/)
3. [Terminal Workflow: tmux, workmux, and Codex](/posts/macos-terminal-workflow/)

I keep one tmux session for work and a separate workmux worktree for each task. Ghostty is the terminal window; tmux windows hold the tasks, and panes hold the agent and shell side by side.

This flow assumes the configuration and status hooks from [Part 1](/posts/macos-terminal-setup/) are in place. Keep [Part 2: Shortcuts](/posts/macos-terminal-shortcuts/) nearby for navigation and copying.

## Daily flow

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

   With the Zsh alias from [Part 1](/posts/macos-terminal-setup/) loaded, `codex` is enough. Give it the task, then use the other shell pane for commands, tests, and reviewing changes. If the agent is already running, continue that session.

4. **Keep the sidebar visible.** Run this from a shell pane inside tmux:

   ```sh
   workmux sidebar --position left on
   ```

   The hooks register the agent and update its status. The identity label tells me which agent is running; the timer shows time since its last status change. For a larger view, run `workmux dashboard` and press **q** when finished.

5. **Review and switch tasks.** Read the agent’s changes, run the project’s checks in the shell pane, and inspect `git diff` before committing. Switch windows with the [shortcuts in Part 2](/posts/macos-terminal-shortcuts/) or use `workmux open` to return to another worktree. Copy terminal output with tmux copy mode or Ghostty’s Shift selection.

6. **Leave and resume.** Press **Ctrl+B**, then **d** to detach. Later, run `tmux new-session -A -s work` again. The session stays available while the tmux server and Mac remain running; it does not survive a reboot by itself.

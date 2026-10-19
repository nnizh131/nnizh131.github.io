---
title: "Terminal Workflow: tmux, workmux, and Codex"
author: Nika Nizharadze
pubDatetime: 2026-10-19T08:02:00Z
modDatetime: 2026-10-19T20:15:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "My daily flow for starting tmux, creating workmux tasks, running Codex, reviewing changes, and returning to work. Part 3 of a three-part series."
---

![Rosé Pine Dawn terminal demo: colorful Starship prompt, Codex completing a task, workmux sidebar, and dashboard diff review.](/terminal-setup/terminal-setup-dawn-colorful.gif)

[Watch the video with playback controls](/terminal-setup/terminal-setup-dawn-colorful.mp4). The demo renders a real tmux, workmux, and Codex session with the Dawn palette and updated prompt colors.

This is a three-part series:

1. [Terminal Setup: Ghostty, tmux, and workmux](/posts/macos-terminal-setup/)
2. [Terminal Shortcuts: Keybindings for Ghostty and tmux](/posts/macos-terminal-shortcuts/)
3. [Terminal Workflow: tmux, workmux, and Codex](/posts/macos-terminal-workflow/)

I keep one tmux session for work and a separate workmux worktree for each task. Ghostty is the terminal window; tmux windows hold the tasks, and panes hold Codex above a shell.

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

   Use a descriptive task name instead of `my-feature`. New worktrees need their own project dependencies and any required local configuration. To return to an existing task, run `workmux open my-feature`.

3. **Start the agent.** In the task’s agent pane, run:

   ```sh
   codex --no-daemon
   ```

   With the Zsh alias from [Part 1](/posts/macos-terminal-setup/) loaded, `codex` is enough. Give it a concrete task, for example: “Replace `src/app.js` with a `greet(name)` function, log `greet("world")`, and run Node to verify.” Use the lower shell pane for commands, tests, and reviewing changes. If the agent is already running, continue that session.

4. **Keep the sidebar visible.** Run this from a shell pane inside tmux:

   ```sh
   workmux sidebar --position left on
   ```

   The hooks register the agent and update its status. **CX** identifies Codex; the timer shows time since its last status change. Run `workmux dashboard --session` for the current session, or omit `--session` for all sessions. **Tab** switches Agents/Worktrees; **d** opens a diff; **q** goes back.

5. **Review and switch tasks.** Read the agent’s changes, run the project’s checks in the shell pane, and inspect `git diff --stat` and `git diff` before committing. In the demo, `node src/app.js` prints `Hello, world!`. Switch windows with the [shortcuts in Part 2](/posts/macos-terminal-shortcuts/) or use `workmux open` to return to another worktree. Copy terminal output with tmux copy mode or Ghostty’s Shift selection.

6. **Leave and resume.** Press **Ctrl+B**, then **d** to detach. Later, run `tmux new-session -A -s work` again. The session stays available while the tmux server and Mac remain running; it does not survive a reboot by itself.

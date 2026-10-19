---
title: "Terminal Shortcuts: Keybindings for Ghostty and tmux"
author: Nika Nizharadze
pubDatetime: 2026-10-19T08:01:00Z
modDatetime: 2026-10-19T20:15:00Z
featured: false
draft: false
tags:
  - macos
  - terminal
  - developer-tools
description: "The Ghostty and tmux shortcuts I use for windows, panes, copying, and persistent sessions. Part 2 of a three-part series."
---

![Rosé Pine Dawn terminal demo: colorful Starship prompt, Codex completing a task, workmux sidebar, and dashboard diff review.](/terminal-setup/terminal-setup-dawn-colorful.gif)

[Watch the video with playback controls](/terminal-setup/terminal-setup-dawn-colorful.mp4). The demo renders a real tmux, workmux, and Codex session with the Dawn palette and updated prompt colors.

This is a three-part series:

1. [Terminal Setup: Ghostty, tmux, and workmux](/posts/macos-terminal-setup/)
2. [Terminal Shortcuts: Keybindings for Ghostty and tmux](/posts/macos-terminal-shortcuts/)
3. [Terminal Workflow: tmux, workmux, and Codex](/posts/macos-terminal-workflow/)

This is the shortcut reference for the [terminal setup in Part 1](/posts/macos-terminal-setup/). The Vi selection keys depend on that post’s tmux bindings, and Shift selection uses its Ghostty configuration.

## Shortcuts

The tmux prefix is **Ctrl+B**. Press it, release it, then press the next key. Copy-mode keys apply after entering copy mode.

| Shortcut                         | Action                                                     |
| -------------------------------- | ---------------------------------------------------------- |
| **Ctrl+B**, then **c**           | Open a new shell window in tmux.                           |
| **Ctrl+B**, then **n** / **p**   | Switch to the next / previous window.                      |
| **Ctrl+B**, then a window number | Switch directly to that window.                            |
| **Ctrl+B**, then **o**           | Move to the next pane, including the sidebar.              |
| **Ctrl+B**, then **z**           | Zoom the current pane; repeat to restore the layout.       |
| **Ctrl+B**, then **[**           | Enter copy mode to navigate scrollback and select text.    |
| **v**, then move, then **y**     | Select and copy in Vi copy mode using the Part 1 bindings. |
| **Shift+drag**, then **Cmd+C**   | Select and copy through Ghostty.                           |
| **Cmd+V**                        | Paste from the macOS clipboard.                            |
| **Ctrl+B**, then **d**           | Detach from tmux while leaving its tasks running.          |
| **Tab** in the dashboard         | Switch between Agents and Worktrees.                       |
| **d** in the dashboard           | Open the selected item’s diff.                             |
| **q** in the workmux dashboard   | Go back from a diff, or close the dashboard.               |

With tmux mouse support enabled, I can also click a pane to focus it and drag pane borders to resize them.

Use **Ctrl+B**, then **c** when you need another shell window. Use `workmux add my-feature` when you need a task with its own Git worktree. Creating a tmux window alone does not create an isolated checkout.

Continue with [Part 3: Daily workflow](/posts/macos-terminal-workflow/) to put these shortcuts into practice.

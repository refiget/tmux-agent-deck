<h1 align="center">tmux-agent-sidebar</h1>

<p align="center">One tmux sidebar that tracks every Claude Code, Codex, and OpenCode pane across every session and window. See status, background shells, prompts, Git state, activity, and worktrees without switching windows.</p>

<p align="center"><img src="website/src/assets/captures/hero.png" alt="tmux-agent-sidebar hero" /></p>

<p align="center">
  Fork of <a href="https://github.com/hiroppy/tmux-agent-sidebar">hiroppy/tmux-agent-sidebar</a> with UI improvements —
  <a href="https://hiroppy.github.io/tmux-agent-sidebar/">upstream docs</a>
</p>

---

## What's different in this fork

The agents panel is now wrapped in a **lazygit-style rounded panel box** instead of being rendered as bare rows.

| Before | After |
|--------|-------|
| ![Before: flat sidebar with no borders](docs/screenshots/before-panel-box.png) | ![After: panel box with rounded border and focus highlight](docs/screenshots/after-panel-box.png) |

**Changes from upstream:**

- **Rounded panel box** — the agents pane is framed with `╭──╮` / `╰──╯` borders (ratatui `BorderType::Rounded`), matching the existing Git panel style
- **Focus-state border color** — the entire border turns accent-blue when the agents panel is focused, dims to gray when focus moves to Git or the terminal — same behavior as lazygit
- **Inner dividers** — `├──┤` separators between the filter row, repo selector row, and agent list, giving each section a distinct visual zone
- **Improved status colors** — response previews and wait-reason rows use colored text instead of saturated background blocks; semantic meaning is carried by glyphs and color, not dark fills
- **Correct group labels** — repo group headers show the tilde-shortened path (`~/.config/tmux`) instead of an uppercased basename

---

## Features

- **Every pane, one view**
  — tracks Claude Code, Codex, and OpenCode panes across all tmux sessions and windows
- **Live metadata**
  — prompts, tool calls, response previews, background shell state, wait reasons, task progress, and subagent trees refresh as the agents work
- **Worktrees, included**
  — spawn a fresh worktree + agent from the sidebar and tear it down — window, worktree, and branch — in one keystroke
- **Desktop notifications**
  — native alerts when an agent finishes, needs permission, or errors out

OpenCode uses a small local plugin bridge instead of per-event hook config. The plugin lives at `.opencode/plugins/tmux-agent-sidebar.js` and can be symlinked as a single file into `~/.config/opencode/plugins/` so it coexists with any existing plugins.

## Requirements

- tmux 3.0+
- [TPM](https://github.com/tmux-plugins/tpm) (or the manual install in [upstream Installation](https://hiroppy.github.io/tmux-agent-sidebar/getting-started/installation/))
- [GitHub CLI](https://cli.github.com/) (optional — required only for PR numbers in the Git tab)

## Quick Start

### 1. Install the plugin

Using [TPM](https://github.com/tmux-plugins/tpm):

```tmux
set -g @plugin 'hiroppy/tmux-agent-sidebar'
```

Reload tmux (`tmux source ~/.tmux.conf`), then press `prefix + I`. The install wizard downloads a pre-built binary or builds from source.

### 2. Wire up the agent hooks

- **Claude Code** — register the plugin inside Claude Code:

  ```sh
  /plugin marketplace add ~/.tmux/plugins/tmux-agent-sidebar
  /plugin install tmux-agent-sidebar@hiroppy
  ```

- **Codex** — open a Codex pane, press `prefix + e`, click the yellow `ⓘ` badge, copy the setup snippet, paste it into the Codex pane.
- **OpenCode** — symlink just the plugin file so your existing `~/.config/opencode/plugins/` contents stay untouched:

  ```sh
  mkdir -p ~/.config/opencode/plugins
  ln -sf ~/.tmux/plugins/tmux-agent-sidebar/.opencode/plugins/tmux-agent-sidebar.js \
    ~/.config/opencode/plugins/tmux-agent-sidebar.js
  ```

### 3. Toggle the sidebar

`prefix + e` toggles the sidebar in the current window, `prefix + E` toggles it everywhere.

## Development

Symlink the plugin directory to your working copy so builds are picked up without copying:

```sh
rm -rf ~/.tmux/plugins/tmux-agent-sidebar
ln -s <path-to-this-repo> ~/.tmux/plugins/tmux-agent-sidebar
cargo build --release
cp target/release/tmux-agent-sidebar bin/tmux-agent-sidebar
```

Toggle the sidebar off → on to pick up the new binary (`bin/` takes priority over `target/release/`).

## License

[MIT](./LICENSE)

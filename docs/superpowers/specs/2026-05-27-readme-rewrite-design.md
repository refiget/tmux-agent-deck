# README Rewrite Design

## Goal
Update `README.md` to reflect the change from a persistent sidebar to a tmux floating window (popup) TUI, and include new visual assets.

## Visual Assets
- `docs/screenshots/floating-popup.png`: Shows the TUI running in a tmux popup.
- `docs/screenshots/bell-notification.png`: Shows the window bell notification in the status bar.

## Structure
1. **Title & Hero**: `tmux-agent-sidebar` with the floating popup screenshot.
2. **Key Feature**: Explain the shift to a floating window TUI.
3. **Notifications**: Highlight the window bell integration with the bell screenshot.
4. **Installation**: Refer users to the [upstream repository](https://github.com/hiroppy/tmux-agent-sidebar) for full installation and setup instructions.
5. **Quick Usage**: Show the `display-popup` keybinding example.

## Proposed Keybinding Snippet
```tmux
bind e display-popup -EE -w 90% -h 90% "tmux-agent-sidebar"
```

## Self-Review
- [x] Includes both images.
- [x] Mentions tmux popup usage.
- [x] Mentions window bell.
- [x] Points to upstream for installation.
- [x] Concise and direct.

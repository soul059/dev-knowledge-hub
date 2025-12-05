# 📟 Tmux Complete Guide

A comprehensive collection of tmux notes, tutorials, and cheatsheets for beginners and advanced users.

![Tmux](https://img.shields.io/badge/tmux-terminal%20multiplexer-green?style=for-the-badge&logo=tmux)
![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)

## 📚 Table of Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | [Introduction](01-introduction.md) | What is tmux, core concepts, installation guide |
| 02 | [Sessions](02-sessions.md) | Session management, attaching, detaching |
| 03 | [Windows](03-windows.md) | Window creation, navigation, management |
| 04 | [Panes](04-panes.md) | Splitting, resizing, layouts, zooming |
| 05 | [Copy Mode](05-copy-mode.md) | Scrollback, searching, copying to clipboard |
| 06 | [Configuration](06-configuration.md) | tmux.conf settings, keybindings, theming |
| 07 | [Plugins](07-plugins.md) | TPM, essential plugins, themes |
| 08 | [Scripting](08-scripting.md) | Automation, shell scripts, tmuxinator |
| 09 | [Cheatsheet](09-cheatsheet.md) | Quick reference card |
| 10 | [Tips & Tricks](10-tips-and-tricks.md) | Pro tips, workflows, debugging |

## 🚀 Quick Start

### Installation

```bash
# Ubuntu/Debian
sudo apt install tmux

# macOS
brew install tmux

# Fedora
sudo dnf install tmux
```

### First Commands

```bash
# Start tmux
tmux

# Create named session
tmux new -s mysession

# Detach (inside tmux)
Ctrl+b d

# Reattach
tmux attach -t mysession
```

## ⌨️ Essential Keybindings

> **Prefix Key**: `Ctrl + b`

| Action | Keys |
|--------|------|
| New window | `Prefix + c` |
| Split vertical | `Prefix + %` |
| Split horizontal | `Prefix + "` |
| Navigate panes | `Prefix + Arrow` |
| Zoom pane | `Prefix + z` |
| Detach | `Prefix + d` |
| List sessions | `Prefix + s` |

## 🎨 Visual Overview

```
┌─────────────────────────────────────────────────┐
│ [session] 0:bash  1:vim*  2:logs        12:00  │ ← Status Bar
├───────────────────────────┬─────────────────────┤
│                           │                     │
│         Pane 1            │       Pane 2        │
│                           │                     │
├───────────────────────────┴─────────────────────┤
│                      Pane 3                     │
└─────────────────────────────────────────────────┘
```

## 📦 Recommended Starter Config

Create `~/.tmux.conf`:

```bash
# Enable mouse
set -g mouse on

# Start index at 1
set -g base-index 1
setw -g pane-base-index 1

# Better splits
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Reload config
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# Vi mode
setw -g mode-keys vi
```

## 🔌 Essential Plugins

Using [TPM (Tmux Plugin Manager)](https://github.com/tmux-plugins/tpm):

```bash
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-yank'
```

## 🤝 Contributing

Contributions are welcome! Feel free to:

- 🐛 Report bugs
- 💡 Suggest new topics
- 📝 Improve documentation
- ⭐ Star this repo!

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [Tmux Official Wiki](https://github.com/tmux/tmux/wiki)
- [Tmux Plugin Manager](https://github.com/tmux-plugins/tpm)
- The amazing tmux community

---

<p align="center">
  Made with ❤️ for terminal lovers
</p>

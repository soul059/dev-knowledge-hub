# Tmux Introduction

## What is Tmux?

**Tmux** (Terminal Multiplexer) is a powerful terminal application that allows you to:

- Run multiple terminal sessions within a single window
- Detach and reattach sessions (keeps processes running in background)
- Split your terminal into multiple panes
- Share sessions with other users

## Why Use Tmux?

| Benefit | Description |
|---------|-------------|
| **Persistence** | Sessions survive disconnections (SSH, network issues) |
| **Productivity** | Multiple windows and panes in one terminal |
| **Remote Work** | Perfect for managing remote servers |
| **Collaboration** | Share terminal sessions with team members |

## Core Concepts

### 1. Server
- Tmux runs as a server in the background
- Manages all sessions, windows, and panes

### 2. Session
- A collection of windows
- Can be detached and reattached
- Named for easy identification

### 3. Window
- Like tabs in a browser
- Each session can have multiple windows
- Only one window visible at a time per client

### 4. Pane
- Splits within a window
- Multiple panes can be visible simultaneously
- Each pane runs its own shell/process

## Visual Hierarchy

```
Tmux Server
    └── Session 1
    │       ├── Window 1
    │       │       ├── Pane 1
    │       │       └── Pane 2
    │       └── Window 2
    │               └── Pane 1
    └── Session 2
            └── Window 1
                    └── Pane 1
```

## Installation

### Ubuntu/Debian
```bash
sudo apt update
sudo apt install tmux
```

### macOS (Homebrew)
```bash
brew install tmux
```

### Fedora/RHEL
```bash
sudo dnf install tmux
```

### Arch Linux
```bash
sudo pacman -S tmux
```

## Verify Installation

```bash
tmux -V
```

## The Prefix Key

- Default prefix: `Ctrl + b`
- All tmux commands start with the prefix
- Press prefix, release, then press the command key
- Example: `Ctrl+b` then `c` creates a new window

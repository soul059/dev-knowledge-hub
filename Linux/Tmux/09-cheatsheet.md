# Tmux Cheatsheet

## Quick Reference Card

> **Prefix Key**: `Ctrl + b` (default)

## Sessions

| Command/Key | Description |
|------------|-------------|
| `tmux` | Start new session |
| `tmux new -s name` | New named session |
| `tmux ls` | List sessions |
| `tmux attach -t name` | Attach to session |
| `tmux kill-session -t name` | Kill session |
| `Prefix + d` | Detach |
| `Prefix + s` | List sessions |
| `Prefix + $` | Rename session |
| `Prefix + (` | Previous session |
| `Prefix + )` | Next session |

## Windows

| Command/Key | Description |
|------------|-------------|
| `Prefix + c` | Create window |
| `Prefix + w` | List windows |
| `Prefix + n` | Next window |
| `Prefix + p` | Previous window |
| `Prefix + 0-9` | Go to window # |
| `Prefix + ,` | Rename window |
| `Prefix + &` | Kill window |
| `Prefix + l` | Last window |
| `Prefix + f` | Find window |

## Panes

| Command/Key | Description |
|------------|-------------|
| `Prefix + %` | Split vertical |
| `Prefix + "` | Split horizontal |
| `Prefix + Arrow` | Navigate panes |
| `Prefix + o` | Cycle panes |
| `Prefix + z` | Zoom/unzoom pane |
| `Prefix + x` | Kill pane |
| `Prefix + q` | Show pane numbers |
| `Prefix + {` | Swap with previous |
| `Prefix + }` | Swap with next |
| `Prefix + !` | Pane to window |
| `Prefix + Space` | Cycle layouts |
| `Prefix + Ctrl+Arrow` | Resize pane |

## Copy Mode (Vi Keys)

| Key | Description |
|-----|-------------|
| `Prefix + [` | Enter copy mode |
| `q` or `Esc` | Exit copy mode |
| `h j k l` | Navigate |
| `/` | Search forward |
| `?` | Search backward |
| `n` / `N` | Next/prev match |
| `Space` or `v` | Start selection |
| `Enter` or `y` | Copy selection |
| `Prefix + ]` | Paste |

## Command Mode

| Key | Description |
|-----|-------------|
| `Prefix + :` | Enter command mode |
| `:new-session` | Create session |
| `:kill-session` | Kill session |
| `:source-file ~/.tmux.conf` | Reload config |

## Miscellaneous

| Command/Key | Description |
|------------|-------------|
| `Prefix + t` | Show clock |
| `Prefix + ?` | List all keybindings |
| `Prefix + :` | Command prompt |

---

## Command Line Quick Reference

```bash
# Sessions
tmux new -s name              # New session
tmux attach -t name           # Attach
tmux ls                       # List
tmux kill-session -t name     # Kill

# Windows  
tmux new-window -n name       # New window
tmux select-window -t :1      # Go to window 1

# Panes
tmux split-window -h          # Split vertical
tmux split-window -v          # Split horizontal

# Send keys
tmux send-keys -t name "cmd" Enter

# Reload config
tmux source ~/.tmux.conf
```

---

## Layout Presets

| Key | Layout |
|-----|--------|
| `Prefix + Alt+1` | Even horizontal |
| `Prefix + Alt+2` | Even vertical |
| `Prefix + Alt+3` | Main horizontal |
| `Prefix + Alt+4` | Main vertical |
| `Prefix + Alt+5` | Tiled |

---

## Essential Config Snippets

```bash
# Mouse support
set -g mouse on

# Vi mode
setw -g mode-keys vi

# Start index at 1
set -g base-index 1

# Reload config
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# Better splits
bind | split-window -h
bind - split-window -v
```

---

## Visual Reference

```
┌─────────────────────────────────────────────┐
│ [session] 0:bash  1:vim*  2:logs     12:00 │  ← Status Bar
├─────────────────────────────────────────────┤
│                    │                        │
│                    │                        │
│      Pane 1        │       Pane 2          │  ← Window
│                    │                        │
│                    │                        │
├─────────────────────────────────────────────┤
│                  Pane 3                     │
└─────────────────────────────────────────────┘

Prefix + %  →  Split Left/Right (Vertical)
Prefix + "  →  Split Top/Bottom (Horizontal)
Prefix + z  →  Zoom pane (toggle fullscreen)
```

---

## Troubleshooting Quick Fixes

| Issue | Solution |
|-------|----------|
| Colors wrong | Add `set -g default-terminal "screen-256color"` |
| Slow escape | Add `set -s escape-time 0` |
| Can't scroll | Use `Prefix + [` for copy mode |
| Mouse not working | Add `set -g mouse on` |

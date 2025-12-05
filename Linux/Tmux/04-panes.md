# Tmux Panes

## What is a Pane?

Panes are subdivisions of a window:
- Split a window horizontally or vertically
- Each pane runs its own shell/process
- Multiple panes visible simultaneously
- Great for side-by-side work

## Creating Panes (Splitting)

| Keybinding | Description |
|------------|-------------|
| `Prefix + %` | Split vertically (left/right) |
| `Prefix + "` | Split horizontally (top/bottom) |

### Visual Reference

```
Prefix + %          Prefix + "
┌─────┬─────┐       ┌───────────┐
│     │     │       │           │
│     │     │       ├───────────┤
│     │     │       │           │
└─────┴─────┘       └───────────┘
```

### Command Line

```bash
# Split current pane vertically
tmux split-window -h

# Split current pane horizontally
tmux split-window -v

# Split with specific size (percentage)
tmux split-window -h -p 30
```

## Navigating Panes

| Keybinding | Description |
|------------|-------------|
| `Prefix + Arrow` | Move to pane in direction |
| `Prefix + o` | Cycle through panes |
| `Prefix + ;` | Toggle to last active pane |
| `Prefix + q` | Show pane numbers (press number to jump) |

### Vim-Style Navigation

Add to `~/.tmux.conf`:
```bash
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
```

## Resizing Panes

| Keybinding | Description |
|------------|-------------|
| `Prefix + Ctrl+Arrow` | Resize in direction (1 cell) |
| `Prefix + Alt+Arrow` | Resize in direction (5 cells) |

### Command Mode

```
:resize-pane -D 10    # Down 10 cells
:resize-pane -U 10    # Up 10 cells
:resize-pane -L 10    # Left 10 cells
:resize-pane -R 10    # Right 10 cells
```

### Mouse Resizing

Enable mouse support:
```bash
set -g mouse on
```

## Pane Layouts

| Keybinding | Description |
|------------|-------------|
| `Prefix + Space` | Cycle through layouts |
| `Prefix + Alt+1` | Even horizontal |
| `Prefix + Alt+2` | Even vertical |
| `Prefix + Alt+3` | Main horizontal |
| `Prefix + Alt+4` | Main vertical |
| `Prefix + Alt+5` | Tiled |

### Layout Examples

```
Even Horizontal     Even Vertical      Main Horizontal
┌───┬───┬───┐      ┌───────────┐      ┌───────────┐
│   │   │   │      ├───────────┤      │           │
│   │   │   │      ├───────────┤      ├─────┬─────┤
│   │   │   │      ├───────────┤      │     │     │
└───┴───┴───┘      └───────────┘      └─────┴─────┘

Main Vertical       Tiled
┌─────┬─────┐      ┌─────┬─────┐
│     ├─────┤      │     │     │
│     ├─────┤      ├─────┼─────┤
│     ├─────┤      │     │     │
└─────┴─────┘      └─────┴─────┘
```

## Zooming Panes

| Keybinding | Description |
|------------|-------------|
| `Prefix + z` | Toggle zoom (fullscreen) current pane |

When zoomed:
- Pane fills entire window
- Other panes hidden but still running
- Press again to restore

## Moving Panes

| Keybinding | Description |
|------------|-------------|
| `Prefix + {` | Swap with previous pane |
| `Prefix + }` | Swap with next pane |
| `Prefix + Ctrl+o` | Rotate panes forward |
| `Prefix + Alt+o` | Rotate panes backward |

### Move to Another Window

```bash
# Break pane into new window
Prefix + !

# Join pane from another window
:join-pane -s :2      # From window 2
:join-pane -s :2.1    # From window 2, pane 1
```

## Closing Panes

| Keybinding | Description |
|------------|-------------|
| `Prefix + x` | Kill current pane (with confirmation) |
| `exit` | Exit the shell (closes pane) |
| `Ctrl + d` | EOF signal (closes pane) |

```bash
# Kill pane by number
tmux kill-pane -t 2
```

## Pane Synchronization

Send same input to all panes:

```bash
# Toggle synchronization
:setw synchronize-panes on
:setw synchronize-panes off

# Or toggle with keybinding
:setw synchronize-panes
```

**Use case**: Run same command on multiple servers

## Pane Borders

Customize in `~/.tmux.conf`:

```bash
# Pane border colors
set -g pane-border-style fg=colour238
set -g pane-active-border-style fg=colour39

# Display pane border status
set -g pane-border-status top
set -g pane-border-format " #{pane_index}: #{pane_current_command} "
```

## Pane Commands Summary

| Command | Description |
|---------|-------------|
| `split-window` | Split current pane |
| `select-pane` | Switch to pane |
| `resize-pane` | Resize pane |
| `swap-pane` | Swap panes |
| `kill-pane` | Close pane |
| `break-pane` | Break pane to new window |
| `join-pane` | Join pane from another window |
| `display-panes` | Show pane numbers |

## Practical Example

Create a development layout:

```bash
# Create main pane
tmux new -s dev -d

# Split for editor (left 70%)
tmux split-window -h -p 30

# Split right side for terminal and logs
tmux select-pane -R
tmux split-window -v

# Select the main editor pane
tmux select-pane -L

# Attach
tmux attach -t dev
```

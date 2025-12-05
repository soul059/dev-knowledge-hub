# Tmux Windows

## What is a Window?

Windows in tmux are like tabs in a browser:
- Each session can have multiple windows
- Only one window is visible at a time
- Windows are numbered starting from 0
- Each window has its own set of panes

## Creating Windows

| Keybinding | Description |
|------------|-------------|
| `Prefix + c` | Create a new window |

### Command Line

```bash
# Create window in new session
tmux new -s dev -n editor

# Create window in existing session
tmux new-window -t dev
tmux new-window -t dev -n logs
```

### Command Mode (Prefix + :)

```
:new-window
:new-window -n window_name
```

## Navigating Windows

| Keybinding | Description |
|------------|-------------|
| `Prefix + n` | Next window |
| `Prefix + p` | Previous window |
| `Prefix + 0-9` | Go to window number 0-9 |
| `Prefix + w` | List all windows (interactive) |
| `Prefix + l` | Toggle to last active window |
| `Prefix + '` | Prompt for window index |

## Renaming Windows

| Method | Command |
|--------|---------|
| Keybinding | `Prefix + ,` |
| Command | `tmux rename-window new_name` |

### Disable Automatic Rename

Add to `~/.tmux.conf`:
```bash
set-option -g allow-rename off
```

## Moving Windows

| Keybinding | Description |
|------------|-------------|
| `Prefix + .` | Move window to another index |

### Command Examples

```bash
# Move window to index 5
tmux move-window -t 5

# Swap windows
tmux swap-window -s 2 -t 1

# Move window to another session
tmux move-window -t other_session:
```

## Closing Windows

| Keybinding | Description |
|------------|-------------|
| `Prefix + &` | Kill current window (with confirmation) |

```bash
# Kill window by index
tmux kill-window -t 2

# Kill window by name
tmux kill-window -t window_name
```

## Window Options

### Set Base Index

Start window numbering from 1 (more intuitive):

```bash
# In ~/.tmux.conf
set -g base-index 1
```

### Renumber Windows

Automatically renumber when a window is closed:

```bash
set -g renumber-windows on
```

## Finding Windows

| Keybinding | Description |
|------------|-------------|
| `Prefix + f` | Find window by name |

## Window Monitoring

### Activity Monitoring

```bash
# Enable activity monitoring
setw -g monitor-activity on
set -g visual-activity on
```

### Silence Monitoring

```bash
# Alert after N seconds of silence
setw -g monitor-silence 30
```

## Linking Windows

Share a window between sessions:

```bash
# Link window from another session
tmux link-window -s source_session:window_index

# Unlink window
tmux unlink-window -t window_index
```

## Window Status Line

The status bar shows window information:

```
[session_name] 0:bash  1:vim* 2:logs-
```

- `*` = Current window
- `-` = Last active window
- `#` = Window has activity
- `!` = Window has bell

## Practical Examples

### Create Project Layout

```bash
# Start new session with named windows
tmux new -s project -n editor -d
tmux new-window -t project -n server
tmux new-window -t project -n logs
tmux attach -t project
```

### Quick Window Switching Script

```bash
# Switch to window by name
tmux select-window -t :editor
```

## Window Commands Summary

| Command | Description |
|---------|-------------|
| `new-window` | Create new window |
| `select-window` | Switch to window |
| `rename-window` | Rename window |
| `kill-window` | Close window |
| `move-window` | Move window |
| `swap-window` | Swap two windows |
| `link-window` | Link window from another session |
| `list-windows` | List all windows |

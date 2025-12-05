# Tmux Sessions

## What is a Session?

A session is the top-level container in tmux that holds windows and panes. Sessions can be:
- Detached (running in background)
- Attached (actively using)
- Named for easy management

## Session Commands

### Starting Sessions

| Command | Description |
|---------|-------------|
| `tmux` | Start a new unnamed session |
| `tmux new` | Start a new unnamed session |
| `tmux new -s name` | Start a new session with name |
| `tmux new -s name -d` | Create session in detached mode |

### Examples

```bash
# Start a new session named "dev"
tmux new -s dev

# Start session with a specific window name
tmux new -s dev -n editor

# Start detached session
tmux new -s background -d
```

## Listing Sessions

```bash
# List all sessions
tmux ls
tmux list-sessions

# Output example:
# dev: 2 windows (created Thu Dec 5 10:00:00 2024)
# main: 1 windows (created Thu Dec 5 09:30:00 2024) (attached)
```

## Attaching to Sessions

```bash
# Attach to last session
tmux attach
tmux a

# Attach to named session
tmux attach -t dev
tmux a -t dev

# Attach and detach other clients
tmux attach -d -t dev
```

## Detaching from Sessions

| Keybinding | Description |
|------------|-------------|
| `Prefix + d` | Detach from current session |
| `Prefix + D` | Choose which client to detach |

```bash
# Detach via command
tmux detach
tmux detach-client
```

## Switching Sessions

| Keybinding | Description |
|------------|-------------|
| `Prefix + s` | List and select sessions |
| `Prefix + (` | Move to previous session |
| `Prefix + )` | Move to next session |
| `Prefix + L` | Switch to last session |

## Renaming Sessions

| Method | Command |
|--------|---------|
| Keybinding | `Prefix + $` |
| Command | `tmux rename-session -t old new` |

## Killing Sessions

```bash
# Kill specific session
tmux kill-session -t name

# Kill all sessions except current
tmux kill-session -a

# Kill all sessions except named one
tmux kill-session -a -t main

# Kill the tmux server (all sessions)
tmux kill-server
```

## Session Within Tmux (Command Mode)

Press `Prefix + :` to enter command mode:

```
# Create new session
:new -s session_name

# Switch to session
:switch-client -t session_name

# Rename current session
:rename-session new_name
```

## Session Groups

Sessions can share windows:

```bash
# Create a new session grouped with existing one
tmux new -s grouped -t existing_session
```

## Practical Tips

### 1. Naming Convention
Use descriptive names for sessions:
- `project-name`
- `server-maintenance`
- `dev-frontend`

### 2. Quick Session Workflow

```bash
# Start work
tmux new -s work

# End of day - detach
# Press: Prefix + d

# Next day - reattach
tmux a -t work
```

### 3. Check if Session Exists

```bash
tmux has-session -t name 2>/dev/null && echo "exists" || echo "not found"
```

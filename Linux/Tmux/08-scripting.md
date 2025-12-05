# Tmux Scripting & Automation

## Command Line Interface

Tmux can be fully controlled from the command line.

### Basic Syntax

```bash
tmux [command] [flags] [arguments]
```

### Targeting

Use `-t` flag to target specific sessions, windows, or panes:

```bash
# Target formats
-t session_name
-t session_name:window_index
-t session_name:window_index.pane_index
-t :window_index
-t :.pane_index
```

## Sending Commands

### send-keys

Send keystrokes to a pane:

```bash
# Send text and press Enter
tmux send-keys -t session:window.pane "echo hello" Enter

# Send Ctrl+C
tmux send-keys -t dev:0.0 C-c

# Send literal text (no special key interpretation)
tmux send-keys -l "literal text"
```

### Run Command in New Window

```bash
tmux new-window -n logs "tail -f /var/log/syslog"
```

## Scripting Examples

### Basic Session Script

```bash
#!/bin/bash
# dev-session.sh

SESSION="development"

# Check if session exists
tmux has-session -t $SESSION 2>/dev/null

if [ $? != 0 ]; then
    # Create new session
    tmux new-session -d -s $SESSION -n editor
    
    # Create additional windows
    tmux new-window -t $SESSION -n terminal
    tmux new-window -t $SESSION -n logs
    
    # Set up editor window
    tmux send-keys -t $SESSION:editor "cd ~/projects && vim" Enter
    
    # Set up logs window
    tmux send-keys -t $SESSION:logs "tail -f /var/log/app.log" Enter
    
    # Select first window
    tmux select-window -t $SESSION:editor
fi

# Attach to session
tmux attach -t $SESSION
```

### Development Layout Script

```bash
#!/bin/bash
# project-layout.sh

SESSION="project"
PROJECT_DIR="$HOME/myproject"

tmux new-session -d -s $SESSION -c $PROJECT_DIR

# Main editor (large left pane)
tmux rename-window -t $SESSION:1 'main'

# Split for terminal
tmux split-window -h -t $SESSION:1 -c $PROJECT_DIR -p 30

# Split right side for two terminals
tmux split-window -v -t $SESSION:1.2 -c $PROJECT_DIR

# Start commands
tmux send-keys -t $SESSION:1.1 'nvim .' Enter
tmux send-keys -t $SESSION:1.2 'npm run dev' Enter
tmux send-keys -t $SESSION:1.3 'npm run test:watch' Enter

# Create additional windows
tmux new-window -t $SESSION -n 'git' -c $PROJECT_DIR
tmux new-window -t $SESSION -n 'db' -c $PROJECT_DIR

# Select main window, first pane
tmux select-window -t $SESSION:1
tmux select-pane -t $SESSION:1.1

tmux attach -t $SESSION
```

### Multi-Server SSH Script

```bash
#!/bin/bash
# multi-ssh.sh

SESSION="servers"
SERVERS=("server1.example.com" "server2.example.com" "server3.example.com")

tmux new-session -d -s $SESSION

# Create a pane for each server
FIRST=true
for SERVER in "${SERVERS[@]}"; do
    if [ "$FIRST" = true ]; then
        FIRST=false
    else
        tmux split-window -h -t $SESSION
    fi
    tmux send-keys -t $SESSION "ssh $SERVER" Enter
done

# Tile the panes
tmux select-layout -t $SESSION tiled

# Synchronize panes
tmux setw -t $SESSION synchronize-panes on

tmux attach -t $SESSION
```

## Tmuxinator

Tmuxinator is a Ruby gem for managing complex tmux sessions.

### Installation

```bash
gem install tmuxinator

# Or with Homebrew
brew install tmuxinator
```

### Create Project

```bash
tmuxinator new project_name
```

### Example Configuration

```yaml
# ~/.config/tmuxinator/project.yml

name: project
root: ~/code/myproject

windows:
  - editor:
      layout: main-vertical
      panes:
        - vim
        - guard

  - server: bundle exec rails server

  - logs: tail -f log/development.log

  - console:
      layout: even-horizontal
      panes:
        - rails console
        - 
```

### Tmuxinator Commands

```bash
# Start session
tmuxinator start project

# Stop session
tmuxinator stop project

# List projects
tmuxinator list

# Edit project
tmuxinator edit project

# Delete project
tmuxinator delete project
```

## Tmuxp

Tmuxp is a Python alternative to Tmuxinator.

### Installation

```bash
pip install tmuxp
```

### Example Configuration

```yaml
# ~/.tmuxp/project.yaml

session_name: project
start_directory: ~/code/myproject

windows:
  - window_name: editor
    layout: main-horizontal
    panes:
      - vim
      - shell_command:
        - npm run watch

  - window_name: server
    panes:
      - npm start

  - window_name: testing
    panes:
      - npm test -- --watch
```

### Tmuxp Commands

```bash
# Load session
tmuxp load project

# Load from file
tmuxp load ~/.tmuxp/project.yaml

# Convert tmuxinator config
tmuxp convert project.yml
```

## Hooks

Execute commands on tmux events.

### Available Hooks

```bash
# After creating a new session
set-hook -g after-new-session 'command'

# After creating a new window
set-hook -g after-new-window 'command'

# After splitting a window
set-hook -g after-split-window 'command'

# When a pane is closed
set-hook -g pane-died 'command'

# When a pane receives focus
set-hook -g pane-focus-in 'command'

# When a window is renamed
set-hook -g window-renamed 'command'
```

### Example Hooks

```bash
# Auto-rename window based on directory
set-hook -g after-new-window 'rename-window "#{b:pane_current_path}"'

# Alert on pane death
set-hook -g pane-died 'display-message "Pane died!"'
```

## Environment Variables

### Built-in Variables

| Variable | Description |
|----------|-------------|
| `TMUX` | Tmux socket path (set when inside tmux) |
| `TMUX_PANE` | Current pane ID |

### Format Variables

Use in status bar or scripts:

| Variable | Description |
|----------|-------------|
| `#{session_name}` | Current session name |
| `#{window_index}` | Current window index |
| `#{window_name}` | Current window name |
| `#{pane_index}` | Current pane index |
| `#{pane_current_path}` | Current pane directory |
| `#{pane_current_command}` | Current command |

### Using in Scripts

```bash
# Get current session
CURRENT_SESSION=$(tmux display-message -p '#S')

# Get current window
CURRENT_WINDOW=$(tmux display-message -p '#I')

# Get pane current path
PANE_PATH=$(tmux display-message -p '#{pane_current_path}')
```

## Conditional Execution

```bash
# Check if inside tmux
if [ -n "$TMUX" ]; then
    echo "Inside tmux"
else
    echo "Outside tmux"
fi

# Check if session exists
if tmux has-session -t mysession 2>/dev/null; then
    tmux attach -t mysession
else
    tmux new -s mysession
fi
```

## Startup Automation

### .bashrc or .zshrc

```bash
# Auto-start tmux
if command -v tmux &> /dev/null && [ -z "$TMUX" ]; then
    tmux attach -t default || tmux new -s default
fi
```

### SSH Auto-attach

```bash
# In .bashrc on remote server
if [[ -n "$SSH_CONNECTION" ]] && [[ -z "$TMUX" ]]; then
    tmux attach -t ssh || tmux new -s ssh
fi
```

## Useful One-Liners

```bash
# Kill all sessions except current
tmux kill-session -a

# Kill all windows except current
tmux kill-window -a

# Get window list
tmux list-windows -F "#{window_index}: #{window_name}"

# Capture pane content
tmux capture-pane -t 0 -p > output.txt

# Save entire scrollback
tmux capture-pane -t 0 -p -S - > scrollback.txt
```

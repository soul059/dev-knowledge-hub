# Tmux Tips & Tricks

## Productivity Tips

### 1. Quick Session Switching

Use `Prefix + s` to get an interactive session list with preview:
- Navigate with arrow keys
- Press Enter to switch
- Press `x` to kill a session

### 2. Command Chaining

Chain multiple commands with `\;`:

```bash
bind R source-file ~/.tmux.conf \; display "Config Reloaded!"
```

### 3. Quick Kill All

```bash
# Kill all sessions
tmux kill-server

# Kill all except current
tmux kill-session -a

# Kill all windows except current
tmux kill-window -a
```

### 4. Work with Current Path

Keep new panes/windows in the same directory:

```bash
bind c new-window -c "#{pane_current_path}"
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
```

### 5. Quick Pane Zoom

`Prefix + z` toggles zoom for focus mode.
- Zoomed pane fills entire window
- Other panes are hidden but running
- Great for temporary focus

## Advanced Tricks

### 1. Synchronized Panes

Run commands on multiple servers simultaneously:

```bash
# Toggle sync
:setw synchronize-panes

# Or bind it
bind S setw synchronize-panes
```

### 2. Quick Capture

Capture pane output to file:

```bash
# Current visible
tmux capture-pane -p > output.txt

# Entire scrollback
tmux capture-pane -pS - > scrollback.txt
```

### 3. Popup Windows (tmux 3.2+)

```bash
# Show popup with command
bind g display-popup -E "lazygit"
bind t display-popup -E "htop"

# Custom size popup
bind p display-popup -w 80% -h 80% -E "vim"
```

### 4. Mark and Swap Panes

```bash
# Mark pane
Prefix + m

# Swap with marked
Prefix + :swap-pane
```

### 5. Break and Join Panes

```bash
# Break pane to new window
Prefix + !

# Join pane from window 2
:join-pane -s :2

# Join specific pane
:join-pane -s :2.1
```

## Configuration Tips

### 1. True Color Support

```bash
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",*256col*:Tc"
```

### 2. Faster Escape

```bash
set -s escape-time 0
```

### 3. Increase History

```bash
set -g history-limit 100000
```

### 4. Focus Events

Required for some Vim plugins:

```bash
set -g focus-events on
```

### 5. Don't Auto-Rename Windows

```bash
set -g allow-rename off
setw -g automatic-rename off
```

## Workflow Tips

### 1. IDE-like Layout

```bash
#!/bin/bash
tmux new-session -d -s ide
tmux split-window -h -p 30
tmux split-window -v -p 50
tmux select-pane -L
tmux attach -t ide

# Result:
# ┌─────────────┬──────┐
# │             │      │
# │   Editor    │ Term │
# │   (70%)     ├──────┤
# │             │ Term │
# └─────────────┴──────┘
```

### 2. Quick SSH Session

```bash
# Create session for server
tmux new -s server -d
tmux send-keys -t server "ssh user@server.com" Enter
tmux attach -t server
```

### 3. Monitoring Layout

```bash
tmux new-session -d -s monitor
tmux send-keys "htop" Enter
tmux split-window -v
tmux send-keys "tail -f /var/log/syslog" Enter
tmux split-window -h
tmux send-keys "watch df -h" Enter
tmux select-layout tiled
tmux attach -t monitor
```

### 4. Development Workflow

```bash
# Morning routine
alias dev='tmux attach -t dev || tmux new -s dev'

# Add to .bashrc/.zshrc
```

## Debugging Tips

### 1. See All Keybindings

```bash
Prefix + ?

# Or command line
tmux list-keys
```

### 2. Check Current Options

```bash
tmux show-options -g
tmux show-window-options -g
```

### 3. Debug Mode

```bash
tmux -v    # Verbose
tmux -vv   # More verbose
```

### 4. Find Conflicts

```bash
# List all key bindings for specific key
tmux list-keys | grep "key-name"
```

## Performance Tips

### 1. Reduce Status Updates

```bash
set -g status-interval 5  # Update every 5 seconds
```

### 2. Simple Status Bar

Complex status bars can slow down:

```bash
set -g status-right "%H:%M"
set -g status-left "#S"
```

### 3. Limit Scrollback

```bash
set -g history-limit 10000  # Reasonable limit
```

## Integration Tips

### 1. FZF Integration

```bash
# Session switcher with fzf
bind s run-shell "tmux list-sessions -F '#{session_name}' | fzf | xargs tmux switch-client -t"
```

### 2. Git Status in Window Name

```bash
set -g status-interval 5
set -g window-status-format '#I:#(cd #{pane_current_path}; basename `git rev-parse --show-toplevel 2>/dev/null || pwd`)'
```

### 3. Vim/Neovim Navigation

Smart pane switching with vim awareness:

```bash
# Vim-tmux-navigator style
is_vim="ps -o state= -o comm= -t '#{pane_tty}' | grep -iqE '^[^TXZ ]+ +(\\S+\\/)?g?(view|n?vim?x?)(diff)?$'"
bind -n C-h if-shell "$is_vim" 'send-keys C-h' 'select-pane -L'
bind -n C-j if-shell "$is_vim" 'send-keys C-j' 'select-pane -D'
bind -n C-k if-shell "$is_vim" 'send-keys C-k' 'select-pane -U'
bind -n C-l if-shell "$is_vim" 'send-keys C-l' 'select-pane -R'
```

## Common Mistakes to Avoid

1. **Nesting tmux sessions** - Use `tmux attach` inside existing session
2. **Forgetting prefix** - All commands need prefix first
3. **Not reloading config** - Remember to source after changes
4. **Huge scrollback** - Limits memory usage
5. **Complex status bar scripts** - Can cause lag

## Quick Fixes

| Problem | Solution |
|---------|----------|
| Weird characters | Check terminal encoding |
| Colors not working | Ensure 256color terminal |
| Can't copy to clipboard | Install xclip/xsel |
| Slow response | Reduce escape-time |
| Mouse not selecting | Check mouse mode settings |

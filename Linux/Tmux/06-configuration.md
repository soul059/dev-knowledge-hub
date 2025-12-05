# Tmux Configuration

## Configuration File

Tmux reads configuration from:
- `~/.tmux.conf` (user config)
- `/etc/tmux.conf` (system-wide)

## Applying Configuration

```bash
# Reload config from within tmux
Prefix + :
:source-file ~/.tmux.conf

# Or from command line
tmux source-file ~/.tmux.conf
```

### Auto-reload Keybinding

```bash
# Add to ~/.tmux.conf
bind r source-file ~/.tmux.conf \; display "Config Reloaded!"
```

## Essential Settings

### Prefix Key

```bash
# Change prefix from Ctrl+b to Ctrl+a
unbind C-b
set -g prefix C-a
bind C-a send-prefix

# Or use both
set -g prefix2 C-a
```

### Mouse Support

```bash
set -g mouse on
```

### Vi Mode

```bash
setw -g mode-keys vi
set -g status-keys vi
```

### History

```bash
set -g history-limit 50000
```

### Indexing

```bash
# Start windows and panes at 1, not 0
set -g base-index 1
setw -g pane-base-index 1

# Renumber windows when one is closed
set -g renumber-windows on
```

### Timing

```bash
# Faster command sequences
set -s escape-time 0

# Increase repeat time for repeatable commands
set -g repeat-time 600

# Display messages longer
set -g display-time 2000
set -g display-panes-time 2000
```

## Key Bindings

### Binding Syntax

```bash
# Basic bind
bind key command

# Bind without prefix
bind -n key command

# Bind in copy mode
bind -T copy-mode-vi key command

# Unbind key
unbind key
```

### Split Panes (Intuitive Keys)

```bash
# Split with | and -
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %
```

### Vim-Style Pane Navigation

```bash
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
```

### Pane Resizing

```bash
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5
```

### Window Navigation

```bash
bind -r C-h previous-window
bind -r C-l next-window
```

## Appearance

### Status Bar Position

```bash
set -g status-position top
# or
set -g status-position bottom
```

### Status Bar Colors

```bash
# Status bar
set -g status-style bg=colour235,fg=colour136

# Window status
setw -g window-status-style fg=colour244,bg=default
setw -g window-status-current-style fg=colour166,bg=default,bright

# Pane borders
set -g pane-border-style fg=colour235
set -g pane-active-border-style fg=colour39

# Message text
set -g message-style bg=colour235,fg=colour166
```

### Status Bar Content

```bash
# Left side
set -g status-left-length 40
set -g status-left "#[fg=green]Session: #S #[fg=yellow]#I #[fg=cyan]#P"

# Right side
set -g status-right-length 60
set -g status-right "#[fg=cyan]%d %b %R"

# Center windows
set -g status-justify centre
```

### 256 Colors

```bash
set -g default-terminal "screen-256color"
set -ga terminal-overrides ",*256col*:Tc"
```

## Copy Mode Configuration

```bash
# Vi mode
setw -g mode-keys vi

# Vi-style copy
bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-selection-and-cancel
bind -T copy-mode-vi r send-keys -X rectangle-toggle

# Copy to system clipboard (Linux)
bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"

# Copy to system clipboard (macOS)
# bind -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"
```

## Activity & Notifications

```bash
# Visual notification of activity
setw -g monitor-activity on
set -g visual-activity on

# Don't auto-rename windows
setw -g automatic-rename off
set -g allow-rename off
```

## Complete Example Configuration

```bash
# ~/.tmux.conf

# ============================================
# === General Settings ===
# ============================================

set -g default-terminal "screen-256color"
set -g history-limit 50000
set -g mouse on
set -s escape-time 0
set -g display-time 2000
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# ============================================
# === Prefix ===
# ============================================

set -g prefix C-a
unbind C-b
bind C-a send-prefix

# ============================================
# === Key Bindings ===
# ============================================

# Reload config
bind r source-file ~/.tmux.conf \; display "Reloaded!"

# Split panes
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"

# Vim-style pane navigation
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# Resize panes
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# ============================================
# === Copy Mode ===
# ============================================

setw -g mode-keys vi
bind -T copy-mode-vi v send-keys -X begin-selection
bind -T copy-mode-vi y send-keys -X copy-selection-and-cancel

# ============================================
# === Status Bar ===
# ============================================

set -g status-position bottom
set -g status-style bg=colour234,fg=colour137
set -g status-left '#[fg=colour233,bg=colour245,bold] #S '
set -g status-right '#[fg=colour233,bg=colour245,bold] %H:%M '
set -g status-left-length 20

setw -g window-status-current-style fg=colour81,bg=colour238,bold
setw -g window-status-current-format ' #I#[fg=colour250]:#[fg=colour255]#W#[fg=colour50]#F '

setw -g window-status-style fg=colour138,bg=colour235
setw -g window-status-format ' #I#[fg=colour237]:#[fg=colour250]#W#[fg=colour244]#F '

# ============================================
# === Panes ===
# ============================================

set -g pane-border-style fg=colour238
set -g pane-active-border-style fg=colour51
```

## Checking Current Settings

```bash
# Show all options
tmux show-options -g

# Show specific option
tmux show-options -g status-style

# Show window options
tmux show-window-options -g
```

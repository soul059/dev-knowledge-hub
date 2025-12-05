# Tmux Copy Mode

## What is Copy Mode?

Copy mode allows you to:
- Scroll through terminal history
- Search for text
- Select and copy text
- Navigate without mouse

## Entering Copy Mode

| Keybinding | Description |
|------------|-------------|
| `Prefix + [` | Enter copy mode |
| `Prefix + PgUp` | Enter copy mode and scroll up |

## Exiting Copy Mode

| Key | Description |
|-----|-------------|
| `q` | Quit copy mode |
| `Escape` | Quit copy mode |
| `Enter` | Copy selection and exit |

## Navigation in Copy Mode

### Basic Movement

| Key | Description |
|-----|-------------|
| `↑ ↓ ← →` | Move cursor |
| `PgUp / PgDn` | Page up/down |
| `g` | Go to top |
| `G` | Go to bottom |

### Vim-Style Movement

| Key | Description |
|-----|-------------|
| `h j k l` | Left, down, up, right |
| `w` | Next word |
| `b` | Previous word |
| `e` | End of word |
| `0` | Start of line |
| `$` | End of line |
| `^` | First non-blank character |
| `Ctrl+u` | Half page up |
| `Ctrl+d` | Half page down |
| `Ctrl+b` | Full page up |
| `Ctrl+f` | Full page down |
| `H` | Top of screen |
| `M` | Middle of screen |
| `L` | Bottom of screen |

## Searching

| Key | Description |
|-----|-------------|
| `/` | Search forward |
| `?` | Search backward |
| `n` | Next search match |
| `N` | Previous search match |

### Example
1. Enter copy mode: `Prefix + [`
2. Press `/`
3. Type search term
4. Press `Enter`
5. Use `n`/`N` to navigate matches

## Selecting Text

### Vi Mode (Recommended)

Add to `~/.tmux.conf`:
```bash
setw -g mode-keys vi
```

| Key | Description |
|-----|-------------|
| `Space` | Start selection |
| `v` | Start selection (vi mode) |
| `V` | Select line (vi mode) |
| `Ctrl+v` | Rectangle selection (vi mode) |
| `Enter` | Copy selection |
| `y` | Copy selection (vi mode) |

### Emacs Mode (Default)

| Key | Description |
|-----|-------------|
| `Ctrl+Space` | Start selection |
| `Alt+w` | Copy selection |

## Copying and Pasting

### Default Workflow

1. `Prefix + [` - Enter copy mode
2. Navigate to text
3. `Space` - Start selection
4. Move to select text
5. `Enter` - Copy and exit
6. `Prefix + ]` - Paste

### Vi Mode Workflow

1. `Prefix + [` - Enter copy mode
2. Navigate to text
3. `v` - Start selection
4. Move to select text
5. `y` - Yank (copy)
6. `Prefix + ]` - Paste

## Enhanced Copy Mode Config

Add to `~/.tmux.conf`:

```bash
# Vi mode
setw -g mode-keys vi

# Vi-style copy bindings
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel
bind-key -T copy-mode-vi r send-keys -X rectangle-toggle

# Mouse selection auto-copies
bind-key -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-selection-and-cancel
```

## System Clipboard Integration

### Linux (xclip)

```bash
# Install xclip
sudo apt install xclip

# Add to ~/.tmux.conf
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"
```

### Linux (xsel)

```bash
# Install xsel
sudo apt install xsel

# Add to ~/.tmux.conf
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xsel --clipboard"
```

### macOS

```bash
# Add to ~/.tmux.conf
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"
bind-key -T copy-mode-vi Enter send-keys -X copy-pipe-and-cancel "pbcopy"

# Paste from clipboard
bind-key p run "pbpaste | tmux load-buffer - && tmux paste-buffer"
```

### WSL (Windows Subsystem for Linux)

```bash
# Add to ~/.tmux.conf
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "clip.exe"
```

## Buffer Management

Tmux maintains multiple paste buffers:

| Command | Description |
|---------|-------------|
| `Prefix + =` | Choose buffer to paste |
| `Prefix + #` | List all buffers |
| `Prefix + -` | Delete most recent buffer |

### Buffer Commands

```bash
# List buffers
tmux list-buffers

# Show buffer content
tmux show-buffer

# Save buffer to file
tmux save-buffer filename.txt

# Load file to buffer
tmux load-buffer filename.txt

# Delete buffer
tmux delete-buffer -b buffer_name
```

## Scrollback History

Configure history limit:

```bash
# In ~/.tmux.conf
set -g history-limit 50000
```

## Mouse Support

Enable mouse for easier copy mode:

```bash
set -g mouse on
```

With mouse enabled:
- Scroll to enter copy mode
- Click and drag to select
- Right-click to paste

## Copy Mode Commands Summary

| Action | Vi Mode | Emacs Mode |
|--------|---------|------------|
| Start selection | `v` or `Space` | `Ctrl+Space` |
| Line selection | `V` | - |
| Rectangle select | `Ctrl+v` | `R` |
| Copy | `y` or `Enter` | `Alt+w` |
| Cancel | `Escape` or `q` | `Escape` or `q` |
| Search forward | `/` | `Ctrl+s` |
| Search backward | `?` | `Ctrl+r` |

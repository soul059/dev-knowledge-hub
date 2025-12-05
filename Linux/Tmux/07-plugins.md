# Tmux Plugins

## Tmux Plugin Manager (TPM)

TPM is the standard plugin manager for tmux.

### Installation

```bash
# Clone TPM
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### Configuration

Add to `~/.tmux.conf`:

```bash
# List of plugins
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# Initialize TPM (keep at bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
```

### TPM Key Bindings

| Keybinding | Description |
|------------|-------------|
| `Prefix + I` | Install plugins |
| `Prefix + U` | Update plugins |
| `Prefix + Alt + u` | Uninstall removed plugins |

## Essential Plugins

### tmux-sensible

Sensible default settings.

```bash
set -g @plugin 'tmux-plugins/tmux-sensible'
```

Features:
- Increase scrollback buffer
- Address vim mode switching delay
- Increase tmux messages display time
- Refresh status more often
- Focus events enabled
- Better key bindings

### tmux-resurrect

Save and restore tmux sessions.

```bash
set -g @plugin 'tmux-plugins/tmux-resurrect'
```

| Keybinding | Description |
|------------|-------------|
| `Prefix + Ctrl+s` | Save session |
| `Prefix + Ctrl+r` | Restore session |

Options:
```bash
# Restore vim sessions
set -g @resurrect-strategy-vim 'session'

# Restore neovim sessions
set -g @resurrect-strategy-nvim 'session'

# Restore pane contents
set -g @resurrect-capture-pane-contents 'on'
```

### tmux-continuum

Automatic saving/restoring with tmux-resurrect.

```bash
set -g @plugin 'tmux-plugins/tmux-continuum'

# Auto-restore on tmux start
set -g @continuum-restore 'on'

# Save interval (minutes)
set -g @continuum-save-interval '15'

# Auto-start tmux on boot (optional)
set -g @continuum-boot 'on'
```

### tmux-yank

Enhanced copy to system clipboard.

```bash
set -g @plugin 'tmux-plugins/tmux-yank'
```

Features:
- Works on Linux, macOS, WSL
- Automatic clipboard detection
- Copy mode enhancements

### tmux-copycat

Advanced search and copy.

```bash
set -g @plugin 'tmux-plugins/tmux-copycat'
```

| Keybinding | Description |
|------------|-------------|
| `Prefix + /` | Regex search |
| `Prefix + Ctrl+f` | Search files |
| `Prefix + Ctrl+g` | Search git status files |
| `Prefix + Alt+h` | Search SHA-1 hashes |
| `Prefix + Ctrl+u` | Search URLs |
| `Prefix + Ctrl+d` | Search numbers |
| `Prefix + Alt+i` | Search IPs |

### tmux-open

Open highlighted selection.

```bash
set -g @plugin 'tmux-plugins/tmux-open'
```

In copy mode:
| Key | Description |
|-----|-------------|
| `o` | Open selection with default program |
| `Ctrl+o` | Open with $EDITOR |
| `S` | Search selection with Google |

### tmux-pain-control

Better pane management bindings.

```bash
set -g @plugin 'tmux-plugins/tmux-pain-control'
```

Bindings added:
- `Prefix + h/j/k/l` - Pane navigation
- `Prefix + H/J/K/L` - Pane resizing
- `Prefix + |` - Vertical split
- `Prefix + -` - Horizontal split
- `Prefix + \` - Vertical split (full height)
- `Prefix + _` - Horizontal split (full width)

### tmux-sessionist

Session management helpers.

```bash
set -g @plugin 'tmux-plugins/tmux-sessionist'
```

| Keybinding | Description |
|------------|-------------|
| `Prefix + g` | Switch to session (prompts) |
| `Prefix + C` | Create new session |
| `Prefix + X` | Kill current session |
| `Prefix + S` | Switch to last session |
| `Prefix + @` | Promote pane to session |

## Theme Plugins

### Dracula Theme

```bash
set -g @plugin 'dracula/tmux'

# Options
set -g @dracula-show-powerline true
set -g @dracula-plugins "cpu-usage ram-usage time"
set -g @dracula-show-left-icon session
```

### Catppuccin Theme

```bash
set -g @plugin 'catppuccin/tmux'

# Flavor: latte, frappe, macchiato, mocha
set -g @catppuccin_flavour 'mocha'
```

### Nord Theme

```bash
set -g @plugin 'arcticicestudio/nord-tmux'
```

### Tokyo Night Theme

```bash
set -g @plugin 'janoamaral/tokyo-night-tmux'
```

### Power Theme

```bash
set -g @plugin 'wfxr/tmux-power'
set -g @tmux_power_theme 'gold'
```

## Utility Plugins

### tmux-fzf

Fuzzy finder integration.

```bash
set -g @plugin 'sainnhe/tmux-fzf'
```

| Keybinding | Description |
|------------|-------------|
| `Prefix + F` | Open tmux-fzf menu |

### tmux-sidebar

Directory tree sidebar.

```bash
set -g @plugin 'tmux-plugins/tmux-sidebar'
```

| Keybinding | Description |
|------------|-------------|
| `Prefix + Tab` | Toggle sidebar |
| `Prefix + Backspace` | Focus sidebar |

### tmux-cpu

CPU and memory info in status bar.

```bash
set -g @plugin 'tmux-plugins/tmux-cpu'

# Add to status bar
set -g status-right '#{cpu_bg_color} CPU: #{cpu_icon} #{cpu_percentage} '
```

### tmux-battery

Battery status in status bar.

```bash
set -g @plugin 'tmux-plugins/tmux-battery'

# Add to status bar
set -g status-right '#{battery_status_bg} #{battery_icon} #{battery_percentage} '
```

### tmux-prefix-highlight

Show when prefix is pressed.

```bash
set -g @plugin 'tmux-plugins/tmux-prefix-highlight'

# Add to status bar
set -g status-right '#{prefix_highlight} | %a %Y-%m-%d %H:%M'
```

## Complete Plugin Setup Example

```bash
# ~/.tmux.conf

# ============================================
# === Plugins ===
# ============================================

set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @plugin 'tmux-plugins/tmux-yank'
set -g @plugin 'tmux-plugins/tmux-pain-control'
set -g @plugin 'catppuccin/tmux'

# ============================================
# === Plugin Settings ===
# ============================================

# Resurrect
set -g @resurrect-capture-pane-contents 'on'

# Continuum
set -g @continuum-restore 'on'
set -g @continuum-save-interval '10'

# Catppuccin
set -g @catppuccin_flavour 'mocha'

# ============================================
# === Initialize TPM ===
# ============================================

run '~/.tmux/plugins/tpm/tpm'
```

## Manual Plugin Installation

Without TPM:

```bash
# Clone plugin
git clone https://github.com/tmux-plugins/tmux-resurrect ~/.tmux/plugins/tmux-resurrect

# Source in tmux.conf
run-shell ~/.tmux/plugins/tmux-resurrect/resurrect.tmux
```

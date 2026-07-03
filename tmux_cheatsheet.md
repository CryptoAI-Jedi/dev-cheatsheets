# tmux Cheatsheet

> Terminal multiplexer: persistent sessions, split panes, and scriptable layouts. The backbone of SSH/VPS work — sessions survive disconnects.

---

## Table of Contents
- [What Is tmux?](#what-is-tmux)
- [Prefix Key](#prefix-key)
- [Sessions](#sessions)
- [Windows (tabs)](#windows-tabs)
- [Panes (splits)](#panes-splits)
- [Copy Mode (scrollback)](#copy-mode-scrollback)
- [Mouse Mode (select, scroll, copy)](#mouse-mode-select-scroll-copy)
- [Buffers (clipboard)](#buffers-clipboard)
- [Command Mode](#command-mode)
- [Synchronize Panes](#synchronize-panes)
- [Scripting tmux (automation)](#scripting-tmux-automation)
- [tmux Bash Script Template](#tmux-bash-script-template)
- [~/.tmux.conf (Config)](#tmuxconf-config)
- [Plugins (TPM)](#plugins-tpm--tmux-plugin-manager)
- [Common Workflows](#common-workflows)
- [Quick Reference: Key Bindings](#quick-reference-key-bindings)

---

## WHAT IS TMUX?

tmux = terminal multiplexer. Run multiple terminal sessions inside one window.
Survives SSH disconnects — sessions persist on the server.

```bash
sudo apt install tmux    # Ubuntu/Debian
sudo pacman -S tmux      # Arch
brew install tmux        # macOS
```

---

## PREFIX KEY

Default prefix: `Ctrl+b` (press, release, then press the next key).
All tmux commands start with the prefix unless run via CLI.
Change prefix to `Ctrl+a` (screen-style) — see [Config](#tmuxconf-config).

---

## SESSIONS

### CLI (outside tmux)

```text
tmux                             → Start new unnamed session
tmux new -s mysession            → Start named session
tmux ls                          → List all sessions
tmux attach                      → Attach to last session
tmux attach -t mysession         → Attach to named session
tmux kill-session -t mysession   → Kill named session
tmux kill-server                 → Kill ALL sessions
```

### Inside tmux (prefix commands)

```text
prefix + $   → Rename current session
prefix + d   → Detach (session stays alive)
prefix + s   → Interactive session list
prefix + (   → Switch to previous session
prefix + )   → Switch to next session
prefix + L   → Switch to last (most recent) session
```

---

## WINDOWS (tabs)

```text
prefix + c     → Create new window
prefix + ,     → Rename current window
prefix + &     → Kill current window (confirm)
prefix + w     → Interactive window list
prefix + n     → Next window
prefix + p     → Previous window
prefix + l     → Last (most recently used) window
prefix + 0-9   → Switch to window by number
prefix + .     → Move window (enter new index)
prefix + f     → Find window by name
```

---

## PANES (splits)

```text
prefix + %   → Split vertically (left | right)
prefix + "   → Split horizontally (top / bottom)
prefix + x   → Kill current pane (confirm)
prefix + z   → Zoom/unzoom pane (toggle fullscreen)
prefix + !   → Break pane into its own window
prefix + {   → Swap pane with previous
prefix + }   → Swap pane with next
prefix + q   → Show pane numbers (press number to jump)
prefix + o   → Cycle through panes
prefix + ;   → Toggle to last active pane
```

### Resize panes

```text
prefix + arrow key   → Resize by 1 cell
prefix + Ctrl+arrow  → Resize by 5 cells
# Or hold prefix and tap arrow repeatedly
```

### Pane layouts (preset)

```text
prefix + Space   → Cycle through layouts
prefix + Alt+1   → Even horizontal
prefix + Alt+2   → Even vertical
prefix + Alt+3   → Main horizontal
prefix + Alt+4   → Main vertical
prefix + Alt+5   → Tiled
```

### Navigate panes

```text
prefix + arrow key   → Move to pane in direction
# With vim-style nav (add to config — see below):
prefix + h/j/k/l     → Left/down/up/right
```

---

## COPY MODE (scrollback)

```text
prefix + [                → Enter copy mode
q                         → Exit copy mode
Arrow keys / PgUp / PgDn  → Scroll
/ then text               → Search forward
? then text               → Search backward
n / N                     → Next / previous match
```

### Vi-style copy (add to config: `setw -g mode-keys vi`)

```text
Space        → Start selection
Enter        → Copy selection (to tmux buffer)
prefix + ]   → Paste tmux buffer
```

### Vi navigation inside copy mode

```text
g / G             → Jump to top / bottom of history
Ctrl+u / Ctrl+d   → Half-page up / down
Ctrl+b / Ctrl+f   → Full page up / down
w / b             → Next / previous word
0 / $             → Start / end of line
```

### Mouse scroll

Scroll wheel scrolls through history if mouse is enabled — see [Mouse Mode](#mouse-mode-select-scroll-copy).

---

## MOUSE MODE (select, scroll, copy)

Enable in `~/.tmux.conf`, then reload (`prefix + :` → `source-file ~/.tmux.conf`, or `prefix + r` if bound):

```bash
set -g mouse on    # Enables scroll, drag-select, pane click, border resize
```

```text
Scroll wheel        → Auto-enters copy mode, scrolls history (q to exit)
Left-click + drag   → Select text (enters copy mode)
Release drag        → Copies selection to tmux buffer + exits copy mode
prefix + ]          → Paste the selection
Middle-click        → Paste most recent tmux buffer
Double-click        → Select word (tmux 3.2+)
Triple-click        → Select line (tmux 3.2+)
Click a pane        → Make it active
Click window name   → Switch window (status bar)
Drag pane border    → Resize panes
Right-click         → Context menu (tmux 3.0+)
```

### Get selections into the SYSTEM clipboard (not just the tmux buffer)

```bash
# Option A — OSC 52: clipboard passes through SSH. Best for VPS work.
# Requires a terminal that supports OSC 52 (kitty, alacritty, WezTerm,
# iTerm2, foot). GNOME Terminal only supports it on VTE 0.76+.
set -g set-clipboard on

# Option B — pipe to a clipboard tool (local sessions only):
bind -T copy-mode-vi MouseDragEnd1Pane send-keys -X copy-pipe-and-cancel "xclip -selection clipboard -in"   # X11
# Wayland: swap the xclip command for "wl-copy"

# Option C — tmux-yank plugin (see PLUGINS) auto-detects the right tool.
```

### Gotchas

```bash
# Releasing a drag auto-copies AND snaps you back to the bottom of
# scrollback. To keep the selection highlighted and yank manually with y:
unbind -T copy-mode-vi MouseDragEnd1Pane

# Shift + drag bypasses tmux entirely → terminal-native select/copy.
# Useful escape hatch, but it grabs pane-border characters if the
# selection crosses a split.

# Mouse selection copies to the TMUX buffer by default — if paste into
# your browser/editor does nothing, you're missing Option A/B/C above.
```

---

## BUFFERS (clipboard)

```text
prefix + ]   → Paste most recent buffer
prefix + =   → List all buffers (interactive)
prefix + -   → Delete most recent buffer
```

### CLI buffer commands

```text
tmux list-buffers          → Show all buffers
tmux show-buffer           → Print most recent buffer
tmux save-buffer file.txt  → Save buffer to file
tmux load-buffer file.txt  → Load file into buffer
tmux set-buffer "text"     → Set buffer manually
tmux paste-buffer          → Paste in current pane
```

---

## COMMAND MODE

`prefix + :` → Enter tmux command prompt

### Useful commands in command mode

```text
:new-window                   → New window
:split-window -h              → Vertical split
:split-window -v              → Horizontal split
:resize-pane -D 10            → Resize pane down 10
:setw synchronize-panes on    → Type in ALL panes simultaneously
:setw synchronize-panes off   → Turn off sync
:source-file ~/.tmux.conf     → Reload config without restart
:kill-session                 → Kill current session
:display-message "hello"      → Show message in status bar
:clock-mode                   → Display clock in pane
```

---

## SYNCHRONIZE PANES

Send the same command to ALL panes in a window — useful for multi-server ops.

```text
prefix + :setw synchronize-panes on
# Run your command (e.g. uptime, git pull) — it types into every pane
prefix + :setw synchronize-panes off
```

---

## SCRIPTING TMUX (automation)

### Start a session with named windows automatically

```bash
tmux new-session -d -s dev -n editor
tmux new-window -t dev -n terminal
tmux new-window -t dev -n monitoring
tmux send-keys -t dev:editor "vim ." Enter
tmux send-keys -t dev:monitoring "htop" Enter
tmux attach -t dev
```

### Split and run commands in panes

```bash
tmux new-session -d -s ops
tmux split-window -h -t ops
tmux split-window -v -t ops
tmux send-keys -t ops:0.0 "ssh server1" Enter
tmux send-keys -t ops:0.1 "ssh server2" Enter
tmux send-keys -t ops:0.2 "tail -f /var/log/syslog" Enter
tmux attach -t ops
```

### Run one-off command in background session

```bash
tmux new-session -d -s bg -x 220 -y 50
tmux send-keys -t bg "bash long_script.sh" Enter
# Detach and check later:
tmux attach -t bg
```

---

## TMUX BASH SCRIPT TEMPLATE

```bash
#!/bin/bash
SESSION="dev"

tmux has-session -t $SESSION 2>/dev/null

if [ $? != 0 ]; then
    tmux new-session -d -s $SESSION -n editor
    tmux new-window -t $SESSION -n shell
    tmux new-window -t $SESSION -n logs
    tmux send-keys -t $SESSION:editor "cd ~/projects && vim" Enter
    tmux send-keys -t $SESSION:shell "cd ~/projects" Enter
    tmux send-keys -t $SESSION:logs "journalctl -f" Enter
    tmux select-window -t $SESSION:editor
fi

tmux attach -t $SESSION
```

---

## ~/.tmux.conf (CONFIG)

Reload config: `prefix + :source-file ~/.tmux.conf` — or add `prefix + r` (see keybinding below).

```bash
# --- Prefix ---
unbind C-b
set-option -g prefix C-a          # Change prefix to Ctrl+a
bind-key C-a send-prefix

# --- General ---
set -g history-limit 50000        # Scrollback buffer size (default 2000 is tiny for log work)
set -g base-index 1               # Windows start at 1 (not 0)
setw -g pane-base-index 1         # Panes start at 1
set -g renumber-windows on        # Auto-renumber on close
set -s escape-time 0              # Remove Esc key delay
set -g display-time 2000          # Status messages linger 2s
set -g focus-events on

# --- Terminal / colors ---
set -g default-terminal "tmux-256color"          # Proper color + italics support
set -ga terminal-overrides ",xterm-256color:Tc"  # True color (24-bit) passthrough

# --- Mouse ---
set -g mouse on                   # Enable mouse (click, scroll, resize) — see MOUSE MODE section

# --- Vi mode ---
setw -g mode-keys vi              # Vi keys in copy mode
bind-key -T copy-mode-vi v send-keys -X begin-selection
# NOTE: y is bound ONCE, in the clipboard section below. A plain
# copy-selection-and-cancel binding here would be silently overridden.

# --- Vim-style pane navigation ---
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R

# --- Splits (more intuitive) ---
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
unbind '"'
unbind %

# --- Reload config ---
bind r source-file ~/.tmux.conf \; display "Config reloaded!"

# --- Resize panes ---
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# --- Status bar ---
set -g status-position bottom
set -g status-style bg=colour235,fg=colour136
set -g status-left "#[fg=colour226][#S] "
set -g status-right "#[fg=colour39]%Y-%m-%d %H:%M"
set -g status-right-length 50
set -g status-left-length 30
setw -g window-status-current-style fg=colour166,bold

# --- Copy to system clipboard ---
# Option A — OSC 52: works over SSH (see MOUSE MODE section for terminal support)
set -g set-clipboard on

# Option B — pipe to clipboard tool (Linux X11 with xclip):
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard -in"
# Wayland: swap the xclip command for "wl-copy"

# macOS:
# bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"
```

---

## PLUGINS (TPM — Tmux Plugin Manager)

### Install TPM

```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

### Add to ~/.tmux.conf

```bash
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'   # Save/restore sessions
set -g @plugin 'tmux-plugins/tmux-continuum'   # Auto-save sessions
set -g @plugin 'tmux-plugins/tmux-yank'        # System clipboard copy

run '~/.tmux/plugins/tpm/tpm'   # Must be LAST line
```

### Plugin management

```text
prefix + I       → Install plugins
prefix + U       → Update plugins
prefix + Alt+u   → Remove plugins
```

### tmux-resurrect usage

```text
prefix + Ctrl+s   → Save session
prefix + Ctrl+r   → Restore session
```

---

## COMMON WORKFLOWS

### SSH + tmux (survive disconnects)

```bash
ssh user@host
tmux new -s work        # Start session on remote
# ... work ...
# Connection drops — SSH back in:
ssh user@host
tmux attach -t work     # Everything still running
```

### Multi-server monitoring

```bash
tmux new -s infra
tmux split-window -h
tmux split-window -v
tmux select-pane -t 0
tmux split-window -v
# Now 4 panes — SSH into different servers in each
```

### Log watching across services

```bash
tmux new -s logs
tmux split-window -h
tmux send-keys "journalctl -fu nginx" Enter
tmux select-pane -t 1
tmux send-keys "journalctl -fu postgresql" Enter
```

---

## QUICK REFERENCE: KEY BINDINGS

| Action            | Key        |
| ----------------- | ---------- |
| New window        | prefix + c |
| Split vertical    | prefix + % |
| Split horizontal  | prefix + " |
| Kill pane         | prefix + x |
| Zoom pane         | prefix + z |
| Detach session    | prefix + d |
| Session list      | prefix + s |
| Window list       | prefix + w |
| Copy mode         | prefix + [ |
| Paste buffer      | prefix + ] |
| Rename window     | prefix + , |
| Rename session    | prefix + $ |
| Command prompt    | prefix + : |
| Show pane numbers | prefix + q |
| Next window       | prefix + n |
| Previous window   | prefix + p |
| Reload config*    | prefix + r |

*requires `bind r` in ~/.tmux.conf

---
*Last Updated: 2026-07*

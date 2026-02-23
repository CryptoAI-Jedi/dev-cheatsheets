# tmux_cheatsheet

---

## WHAT IS TMUX?
tmux = terminal multiplexer. Run multiple terminal sessions inside one window.
Survives SSH disconnects — sessions persist on the server.
Install: sudo apt install tmux / sudo pacman -S tmux / brew install tmux

---

## PREFIX KEY
Default prefix: Ctrl+b  (press, release, then press the next key)
All tmux commands start with the prefix unless run via CLI.
Change prefix to Ctrl+a (screen-style) — see Config section.

---

## SESSIONS
# CLI (outside tmux)
tmux                                → Start new unnamed session
tmux new -s mysession               → Start named session
tmux ls                             → List all sessions
tmux attach                         → Attach to last session
tmux attach -t mysession            → Attach to named session
tmux kill-session -t mysession      → Kill named session
tmux kill-server                    → Kill ALL sessions

# Inside tmux (prefix commands)
prefix + $                          → Rename current session
prefix + d                          → Detach (session stays alive)
prefix + s                          → Interactive session list
prefix + (                          → Switch to previous session
prefix + )                          → Switch to next session
prefix + L                          → Switch to last (most recent) session

---

## WINDOWS (tabs)
prefix + c                          → Create new window
prefix + ,                          → Rename current window
prefix + &                          → Kill current window (confirm)
prefix + w                          → Interactive window list
prefix + n                          → Next window
prefix + p                          → Previous window
prefix + l                          → Last (most recently used) window
prefix + 0-9                        → Switch to window by number
prefix + .                          → Move window (enter new index)
prefix + f                          → Find window by name

---

## PANES (splits)
prefix + %                          → Split vertically (left | right)
prefix + "                          → Split horizontally (top / bottom)
prefix + x                          → Kill current pane (confirm)
prefix + z                          → Zoom/unzoom pane (toggle fullscreen)
prefix + !                          → Break pane into its own window
prefix + {                          → Swap pane with previous
prefix + }                          → Swap pane with next
prefix + q                          → Show pane numbers (press number to jump)
prefix + o                          → Cycle through panes
prefix + ;                          → Toggle to last active pane

# Resize panes
prefix + arrow key                  → Resize by 1 cell
prefix + Ctrl+arrow                 → Resize by 5 cells
# Or hold prefix and tap arrow repeatedly

# Pane layouts (preset)
prefix + Space                      → Cycle through layouts
prefix + Alt+1                      → Even horizontal
prefix + Alt+2                      → Even vertical
prefix + Alt+3                      → Main horizontal
prefix + Alt+4                      → Main vertical
prefix + Alt+5                      → Tiled

# Navigate panes
prefix + arrow key                  → Move to pane in direction
# With vim-style nav (add to config — see below):
prefix + h/j/k/l                    → Left/down/up/right

---

## COPY MODE (scrollback)
prefix + [                          → Enter copy mode
q                                   → Exit copy mode
Arrow keys / PgUp / PgDn            → Scroll
/ then text                         → Search forward
? then text                         → Search backward
n / N                               → Next / previous match

# Vi-style copy (add to config: setw -g mode-keys vi)
Space                               → Start selection
Enter                               → Copy selection (to tmux buffer)
prefix + ]                          → Paste tmux buffer

# Mouse scroll (if mouse enabled — see config)
Scroll wheel                        → Scroll through history

---

## BUFFERS (clipboard)
prefix + ]                          → Paste most recent buffer
prefix + =                          → List all buffers (interactive)
prefix + -                          → Delete most recent buffer

# CLI buffer commands
tmux list-buffers                   → Show all buffers
tmux show-buffer                    → Print most recent buffer
tmux save-buffer file.txt           → Save buffer to file
tmux load-buffer file.txt           → Load file into buffer
tmux set-buffer "text"              → Set buffer manually
tmux paste-buffer                   → Paste in current pane

---

## COMMAND MODE
prefix + :                          → Enter tmux command prompt

# Useful commands in command mode
:new-window                         → New window
:split-window -h                    → Vertical split
:split-window -v                    → Horizontal split
:resize-pane -D 10                  → Resize pane down 10
:setw synchronize-panes on          → Type in ALL panes simultaneously
:setw synchronize-panes off         → Turn off sync
:source-file ~/.tmux.conf           → Reload config without restart
:kill-session                       → Kill current session
:display-message "hello"            → Show message in status bar
:clock-mode                         → Display clock in pane

---

## SYNCHRONIZE PANES
# Send same command to ALL panes in a window — useful for multi-server ops
prefix + :setw synchronize-panes on
# Run your command (e.g. uptime, git pull)
prefix + :setw synchronize-panes off

---

## SCRIPTING TMUX (automation)
# Start a session with named windows automatically
tmux new-session -d -s dev -n editor
tmux new-window -t dev -n terminal
tmux new-window -t dev -n monitoring
tmux send-keys -t dev:editor "vim ." Enter
tmux send-keys -t dev:monitoring "htop" Enter
tmux attach -t dev

# Split and run commands in panes
tmux new-session -d -s ops
tmux split-window -h -t ops
tmux split-window -v -t ops
tmux send-keys -t ops:0.0 "ssh server1" Enter
tmux send-keys -t ops:0.1 "ssh server2" Enter
tmux send-keys -t ops:0.2 "tail -f /var/log/syslog" Enter
tmux attach -t ops

# Run one-off command in background session
tmux new-session -d -s bg -x 220 -y 50
tmux send-keys -t bg "bash long_script.sh" Enter
# Detach and check later
tmux attach -t bg

---

## TMUX BASH SCRIPT TEMPLATE
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

---

## ~/.tmux.conf (CONFIG)
# Reload config: prefix + :source-file ~/.tmux.conf
# Or add: prefix + r (see keybinding below)

# --- Prefix ---
unbind C-b
set-option -g prefix C-a           # Change prefix to Ctrl+a
bind-key C-a send-prefix

# --- General ---
set -g history-limit 10000         # Scrollback buffer size
set -g base-index 1                # Windows start at 1 (not 0)
setw -g pane-base-index 1          # Panes start at 1
set -g renumber-windows on         # Auto-renumber on close
set -s escape-time 0               # Remove Esc key delay
set -g display-time 2000           # Status messages linger 2s
set -g focus-events on

# --- Mouse ---
set -g mouse on                    # Enable mouse (click, scroll, resize)

# --- Vi mode ---
setw -g mode-keys vi               # Vi keys in copy mode
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel

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

# --- Copy to system clipboard (Linux with xclip) ---
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard -in"

# --- Copy to system clipboard (macOS) ---
# bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "pbcopy"

---

## PLUGINS (TPM — Tmux Plugin Manager)
# Install TPM
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# Add to ~/.tmux.conf
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'    # Save/restore sessions
set -g @plugin 'tmux-plugins/tmux-continuum'    # Auto-save sessions
set -g @plugin 'tmux-plugins/tmux-yank'         # System clipboard copy

run '~/.tmux/plugins/tpm/tpm'                   # Must be LAST line

# Install plugins: prefix + I
# Update plugins:  prefix + U
# Remove plugins:  prefix + alt+u

# tmux-resurrect usage
prefix + Ctrl+s                    → Save session
prefix + Ctrl+r                    → Restore session

---

## COMMON WORKFLOWS

### SSH + tmux (survive disconnects)
ssh user@host
tmux new -s work                   → Start session on remote
# ... work ...
# Connection drops — SSH back in:
ssh user@host
tmux attach -t work                → Everything still running

### Multi-server monitoring
tmux new -s infra
tmux split-window -h
tmux split-window -v
tmux select-pane -t 0
tmux split-window -v
# Now 4 panes — SSH into different servers in each

### Log watching across services
tmux new -s logs
tmux split-window -h
tmux send-keys "journalctl -fu nginx" Enter
tmux select-pane -t 1
tmux send-keys "journalctl -fu postgresql" Enter

---

## QUICK REFERENCE: KEY BINDINGS

| Action                  | Key            |
|-------------------------|----------------|
| New window              | prefix + c     |
| Split vertical          | prefix + %     |
| Split horizontal        | prefix + "     |
| Kill pane               | prefix + x     |
| Zoom pane               | prefix + z     |
| Detach session          | prefix + d     |
| Session list            | prefix + s     |
| Window list             | prefix + w     |
| Copy mode               | prefix + [     |
| Paste buffer            | prefix + ]     |
| Rename window           | prefix + ,     |
| Rename session          | prefix + $     |
| Command prompt          | prefix + :     |
| Show pane numbers       | prefix + q     |
| Next window             | prefix + n     |
| Previous window         | prefix + p     |
| Reload config*          | prefix + r     |

*requires bind r in ~/.tmux.conf

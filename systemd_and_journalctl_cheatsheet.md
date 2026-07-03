# systemd & journalctl Cheatsheet

> Service manager + logging for every modern Linux box. Run agents as supervised services that restart on failure, survive reboots, and log to a queryable journal — no more `nohup` and scattered logfiles.

---

## Table of Contents
- [What Is systemd?](#what-is-systemd)
- [Service Lifecycle (systemctl)](#service-lifecycle-systemctl)
- [Inspecting Units](#inspecting-units)
- [Unit Files (writing services)](#unit-files-writing-services)
- [User Services (no root)](#user-services-no-root)
- [journalctl Basics](#journalctl-basics)
- [journalctl Filtering](#journalctl-filtering)
- [Journal Maintenance](#journal-maintenance)
- [Timers (cron replacement)](#timers-cron-replacement)
- [Analysis & Troubleshooting](#analysis--troubleshooting)
- [Common Workflows](#common-workflows)
- [Tips & Gotchas](#tips--gotchas)

---

## WHAT IS SYSTEMD?

Init system + service manager (PID 1). Everything it manages is a **unit**: `.service`, `.timer`, `.socket`, `.target`, `.mount`. `systemctl` controls units; `journalctl` reads their logs.

---

## SERVICE LIFECYCLE (systemctl)

```bash
sudo systemctl start nginx          # Start now
sudo systemctl stop nginx           # Stop now
sudo systemctl restart nginx        # Stop + start
sudo systemctl reload nginx         # Re-read config WITHOUT dropping connections
sudo systemctl reload-or-restart nginx   # Reload if supported, else restart
systemctl status nginx              # State, PID, recent log lines
```

```bash
sudo systemctl enable nginx         # Start at boot
sudo systemctl enable --now nginx   # Enable + start in one shot
sudo systemctl disable nginx        # Don't start at boot (doesn't stop it now)
sudo systemctl mask nginx           # Block ALL starts, even manual/dependency
sudo systemctl unmask nginx
```

```bash
systemctl is-active nginx           # active / inactive / failed (scriptable)
systemctl is-enabled nginx          # enabled / disabled / masked
systemctl is-failed nginx
```

---

## INSPECTING UNITS

```bash
systemctl list-units --type=service                  # Loaded services
systemctl list-units --type=service --state=running  # Only running
systemctl list-unit-files --type=service             # ALL installed + enable state
systemctl --failed                                   # Failed units (check after boot)
```

```bash
systemctl cat myagent               # Show unit file + drop-in overrides
systemctl show myagent              # Every property (add -p MainPID,MemoryCurrent)
systemctl list-dependencies myagent # What it pulls in / waits for
```

---

## UNIT FILES (writing services)

```text
/etc/systemd/system/     → Your units + overrides (wins)
/lib/systemd/system/     → Package-installed units (don't edit)
/run/systemd/system/     → Runtime-generated
```

### Long-running service template (Python agent on a VPS)

```bash
# /etc/systemd/system/myagent.service
[Unit]
Description=Hermes agent
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=hermes
WorkingDirectory=/opt/agent
EnvironmentFile=/opt/agent/.env
Environment=PYTHONUNBUFFERED=1        # Logs stream to journal immediately
ExecStart=/opt/agent/venv/bin/python main.py
Restart=always                        # Agents: always. Crash OR clean exit → back up
RestartSec=5

# Optional caps — contain a runaway agent:
MemoryMax=512M
CPUQuota=50%
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload          # REQUIRED after creating/editing unit files
sudo systemctl enable --now myagent
```

### Edit an existing unit (survives package updates)

```bash
sudo systemctl edit myagent           # Drop-in override (only the keys you change)
sudo systemctl edit --full myagent    # Edit a full copy
sudo systemctl revert myagent         # Delete overrides, back to stock
```

---

## USER SERVICES (no root)

Run services under your own account — units live in `~/.config/systemd/user/`.

```bash
systemctl --user daemon-reload
systemctl --user enable --now myagent
systemctl --user status myagent
journalctl --user -u myagent -f
```

```bash
# CRITICAL for VPS agents: user services die when you log out, unless:
loginctl enable-linger $USER          # Services persist with no active session
loginctl show-user $USER | grep Linger   # Verify
```

```text
⚠️ User units use `WantedBy=default.target` in [Install], not multi-user.target.
```

---

## JOURNALCTL BASICS

```bash
journalctl -u myagent                 # All logs for a unit
journalctl -u myagent -f              # Follow (tail -f equivalent)
journalctl -u myagent -n 100          # Last 100 lines
journalctl -u myagent -e              # Open pager at the END
journalctl -xeu myagent               # End + explanatory text — the debug default
```

```bash
journalctl --since "1 hour ago"
journalctl --since today
journalctl --since "2026-07-01 00:00" --until "2026-07-02 12:00"
journalctl -b                         # Current boot only
journalctl -b -1                      # Previous boot (post-mortem)
journalctl --list-boots
journalctl -k                         # Kernel messages (dmesg equivalent)
```

```text
💡 Reading the system journal without sudo requires membership in the
adm or systemd-journal group: sudo usermod -aG adm $USER
```

---

## JOURNALCTL FILTERING

```bash
journalctl -p err                     # Priority err and worse
journalctl -p warning..err            # Priority range
journalctl -u myagent -g "timeout"    # Grep within results (regex)
journalctl -u nginx -u myagent        # Multiple units, interleaved
journalctl -t sshd                    # By syslog identifier (tag)
journalctl _PID=1234                  # By field (any journal field works)
journalctl -r                         # Newest first
```

```text
Priorities: emerg(0) alert(1) crit(2) err(3) warning(4) notice(5) info(6) debug(7)
```

### Output formats

```bash
journalctl -u myagent -o cat          # Message text only (pipe-friendly)
journalctl -u myagent -o short-iso    # ISO timestamps
journalctl -u myagent -o json-pretty  # Full structured record
journalctl -u myagent --no-pager      # For scripts/redirects
```

---

## JOURNAL MAINTENANCE

```bash
journalctl --disk-usage               # How much space logs consume
sudo journalctl --vacuum-size=500M    # Shrink to 500M
sudo journalctl --vacuum-time=2weeks  # Drop entries older than 2 weeks
sudo journalctl --verify              # Check journal integrity
```

```bash
# /etc/systemd/journald.conf — make limits permanent:
Storage=persistent                    # Survive reboots (needs /var/log/journal)
SystemMaxUse=1G

# Then:
sudo systemctl restart systemd-journald
```

```text
⚠️ If logs vanish on reboot: Storage=auto with no /var/log/journal dir
means volatile-only. Fix: sudo mkdir -p /var/log/journal && sudo
systemd-tmpfiles --create --prefix /var/log/journal
```

---

## TIMERS (cron replacement)

A timer unit triggers a matching service unit. Wins over cron: logs in the journal, `Persistent=` catch-up for missed runs, per-job resource caps.

```bash
# /etc/systemd/system/backup.service
[Unit]
Description=Nightly backup

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```

```bash
# /etc/systemd/system/backup.timer
[Unit]
Description=Run backup nightly

[Timer]
OnCalendar=*-*-* 03:00:00
Persistent=true                       # Run at next boot if the box was off at 3am
RandomizedDelaySec=300                # Jitter — don't stampede shared resources

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now backup.timer   # Enable the TIMER, not the service
systemctl list-timers                      # Next/last run for all timers
```

### OnCalendar syntax

```text
OnCalendar=hourly                 → Top of every hour
OnCalendar=daily                  → Midnight
OnCalendar=*:0/15                 → Every 15 minutes
OnCalendar=Mon..Fri 09:00         → Weekdays at 9am
OnCalendar=*-*-01 06:00:00        → 1st of the month, 6am
```

```bash
systemd-analyze calendar "Mon..Fri 09:00"   # Validate + show next trigger times
systemd-run --on-active=30s /usr/bin/touch /tmp/x   # Ad-hoc transient timer
```

---

## ANALYSIS & TROUBLESHOOTING

```bash
systemd-analyze                       # Total boot time
systemd-analyze blame                 # Slowest units at boot
systemd-analyze critical-chain        # Boot dependency chain
systemd-analyze verify myagent.service   # Lint a unit file before deploying
sudo systemctl reset-failed           # Clear failed state (also resets restart counters)
```

---

## COMMON WORKFLOWS

### Deploy a Python agent as a service

```bash
sudo cp myagent.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now myagent
systemctl status myagent
journalctl -fu myagent                # Watch it come up
```

### Debug a failing service

```bash
systemctl status myagent              # What state, what exit code?
journalctl -xeu myagent               # Why — end of log + explanations
systemctl cat myagent                 # Is the unit what you think it is?
sudo systemd-analyze verify /etc/systemd/system/myagent.service
sudo systemctl daemon-reload && sudo systemctl restart myagent
```

### Multi-service log watching (pairs with tmux panes)

```bash
journalctl -fu nginx
journalctl -fu myagent -p warning     # Warnings and worse only
journalctl -f -u nginx -u myagent     # Or interleaved in one pane
```

---

## TIPS & GOTCHAS

- **Edited a unit and nothing changed** — you skipped `daemon-reload`. systemd caches unit files; reload, then restart.
- **`enable` ≠ `start`** — enable wires boot-time startup only. Use `enable --now` to do both; symmetric trap: `disable` doesn't stop a running service.
- **`ExecStart` needs absolute paths** — no shell, no `$PATH`, no pipes/redirection. Need shell features? `ExecStart=/bin/bash -c '...'` (sparingly).
- **`Restart=on-failure` won't resurrect a clean exit** — an agent that exits 0 stays down. For always-on agents use `Restart=always`.
- **Rapid-crash loops hit the start limit** — 5 failures in 10s and systemd gives up (`start-limit-hit`). `systemctl reset-failed myagent` clears it; fix the crash, or set `StartLimitIntervalSec=0` for die-hard agents.
- **User services vanish after you disconnect** — no linger, no service. `loginctl enable-linger $USER` is step zero for user-level agents on a VPS.
- **`EnvironmentFile` is not shell** — no `export`, no quotes needed, no variable expansion. `KEY=value` lines only.
- **Python logs missing from the journal** — stdout is block-buffered without a TTY. `Environment=PYTHONUNBUFFERED=1` (or `python -u`).
- **`journalctl --since` uses local time** — on UTC servers, "today" starts at UTC midnight, not yours.

---
*Last Updated: 2026-07*

# systemd_journalctl_cheatsheet

# SYSTEMCTL — SERVICE MANAGEMENT

---

systemctl start service            → Start a service
systemctl stop service             → Stop a service
systemctl restart service          → Stop then start
systemctl reload service           → Reload config (no downtime)
systemctl enable service           → Enable at boot
systemctl disable service          → Disable at boot
systemctl enable --now service     → Enable AND start immediately
systemctl disable --now service    → Disable AND stop immediately
systemctl status service           → Status, recent logs, PID
systemctl is-active service        → Returns "active" or "inactive"
systemctl is-enabled service       → Returns "enabled" or "disabled"
systemctl is-failed service        → Returns "failed" or "active"

---

## SYSTEMCTL — LISTING & INSPECTION

systemctl list-units               → All active units
systemctl list-units --type=service        → Active services only
systemctl list-units --type=service --all  → All services (inc. inactive)
systemctl list-units --state=failed        → Failed units
systemctl list-unit-files                  → All installed unit files
systemctl list-unit-files --type=service   → Installed services
systemctl list-timers                      → All systemd timers
systemctl list-dependencies service        → Dependency tree
systemctl cat service                      → Show unit file contents
systemctl show service                     → All properties (key=value)
systemctl show -p Restart service          → Show specific property

---

## SYSTEMCTL — SYSTEM STATE

systemctl daemon-reload            → Reload unit files after edits
systemctl reset-failed             → Clear failed status for all units
systemctl reset-failed service     → Clear failed status for one unit
systemctl mask service             → Prevent service from ever starting
systemctl unmask service           → Re-enable masked service
systemctl isolate multi-user.target    → Switch runlevel (no GUI)
systemctl isolate graphical.target     → Switch to GUI runlevel
systemctl get-default              → Show default target
systemctl set-default multi-user.target

# Power

systemctl poweroff                 → Shutdown
systemctl reboot                   → Reboot
systemctl suspend                  → Suspend to RAM
systemctl hibernate                → Suspend to disk

---

## UNIT FILE LOCATIONS

/lib/systemd/system/               → Package-installed unit files (don't edit)
/etc/systemd/system/               → Admin overrides (edit here)
~/.config/systemd/user/            → User-level unit files

# Override without replacing (preferred)

systemctl edit service             → Creates drop-in override
systemctl edit --full service      → Edit full copy of unit file
/etc/systemd/system/service.d/override.conf

---

## WRITING A CUSTOM SERVICE UNIT

# /etc/systemd/system/myapp.service

[Unit]
Description=My Application
After=network.target
Requires=postgresql.service

[Service]
Type=simple
User=www-data
Group=www-data
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5s
StandardOutput=journal
StandardError=journal
Environment="APP_ENV=production"
EnvironmentFile=/etc/myapp/myapp.env

[Install]
WantedBy=multi-user.target

# After creating/editing:

systemctl daemon-reload
systemctl enable --now myapp

---

## SERVICE TYPES

Type=simple       → Default; ExecStart is the main process
Type=exec         → Like simple, waits for exec to succeed
Type=forking      → Process forks to background (traditional daemons)
Type=oneshot      → Runs once then exits (use RemainAfterExit=yes)
Type=notify       → Process signals ready via sd_notify()
Type=idle         → Delays start until all jobs are dispatched

---

## WRITING A TIMER UNIT (systemd cron)

# /etc/systemd/system/mybackup.timer

[Unit]
Description=Run backup every day at 2am

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true                   → Run missed jobs on next boot
Unit=mybackup.service

[Install]
WantedBy=timers.target

# Activate timer

systemctl enable --now mybackup.timer
systemctl list-timers              → Verify timer is active

# OnCalendar examples

OnCalendar=daily                   → Midnight every day
OnCalendar=hourly                  → Top of every hour
OnCalendar=weekly                  → Monday midnight
OnCalendar=Mon..Fri 09:00          → Weekdays at 9am
OnCalendar=*:0/15                  → Every 15 minutes
OnBootSec=5min                     → 5 minutes after boot
OnUnitActiveSec=1h                 → Every hour after last run

---

## JOURNALCTL — LOG VIEWING

journalctl                         → All logs (oldest first)
journalctl -r                      → Reverse (newest first)
journalctl -f                      → Follow live logs (like tail -f)
journalctl -e                      → Jump to end of logs
journalctl -n 50                   → Last 50 lines
journalctl -n 100 -r               → Last 100 lines, newest first

---

## JOURNALCTL — FILTERING

journalctl -u service              → Logs for specific unit
journalctl -u nginx -u postgresql  → Logs for multiple units
journalctl -f -u service           → Follow specific service
journalctl -b                      → Logs since last boot
journalctl -b -1                   → Logs from previous boot
journalctl -b -2                   → Logs from 2 boots ago
journalctl --list-boots            → List all recorded boots
journalctl --since "1 hour ago"    → Last hour of logs
journalctl --since "2024-01-01"    → Logs from date
journalctl --since "09:00" --until "10:00"
journalctl -p err                  → Error priority and above
journalctl -p warning              → Warning priority and above
journalctl -p 0..3                 → emerg/alert/crit/err only

# Priority levels (low to high severity number)

0=emerg 1=alert 2=crit 3=err 4=warning 5=notice 6=info 7=debug

---

## JOURNALCTL — ADVANCED FILTERING

journalctl _PID=1234               → Logs from specific PID
journalctl _UID=1000               → Logs from specific user ID
journalctl _SYSTEMD_UNIT=nginx.service  → Alternative unit filter
journalctl /usr/bin/python3        → Logs from executable path
journalctl -k                      → Kernel logs only (dmesg)
journalctl -t myapp                → Logs from syslog identifier

---

## JOURNALCTL — OUTPUT FORMATS

journalctl -o short                → Default format
journalctl -o short-iso            → ISO timestamps
journalctl -o json                 → JSON (one object per line)
journalctl -o json-pretty          → Pretty JSON
journalctl -o cat                  → Message only (no metadata)
journalctl -o verbose              → All fields
journalctl -u nginx --no-pager     → Disable pager (pipe-friendly)
journalctl -u nginx | grep "error" → Pipe to grep

---

## JOURNALCTL — DISK USAGE & CLEANUP

journalctl --disk-usage            → Show journal disk usage
journalctl --vacuum-size=500M      → Keep max 500MB of logs
journalctl --vacuum-time=7d        → Keep logs from last 7 days
journalctl --vacuum-files=5        → Keep only 5 journal files

# Persistent journal config

# /etc/systemd/journald.conf

[Journal]
Storage=persistent                 → Save logs across reboots
SystemMaxUse=500M                  → Max disk usage
MaxRetentionSec=1month             → Max age

systemctl restart systemd-journald → Apply config changes

---

## COMMON TROUBLESHOOTING WORKFLOW

# 1. Check failed services

systemctl list-units --state=failed

# 2. Inspect failed service

systemctl status myservice

# 3. Get full logs for failed service

journalctl -u myservice -n 100 --no-pager

# 4. Get logs since last boot

journalctl -u myservice -b

# 5. Check last crash/reboot

journalctl -b -1 -p err

# 6. Restart and follow live

systemctl restart myservice && journalctl -f -u myservice
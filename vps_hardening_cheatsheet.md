# VPS Hardening Cheatsheet

> Lock down a fresh Linux VPS: firewall, brute-force response, automatic patching, and periodic audits. Completes the perimeter with the [SSH](ssh_cheatsheet.md), [SSH/GPG Keys](ssh_gpg_keys_cheatsheet.md), and [nginx](nginx_cheatsheet.md) sheets.

---

## Table of Contents
- [Hardening Order (First Hour)](#hardening-order-first-hour)
- [Users & Privileges](#users--privileges)
- [sshd Lockdown (recap)](#sshd-lockdown-recap)
- [UFW (firewall)](#ufw-firewall)
- [fail2ban](#fail2ban)
- [Automatic Security Updates](#automatic-security-updates)
- [Tailnet-First Pattern (Tailscale)](#tailnet-first-pattern-tailscale)
- [Audit & Monitoring](#audit--monitoring)
- [Common Workflows](#common-workflows)
- [Tips & Gotchas](#tips--gotchas)

---

## HARDENING ORDER (FIRST HOUR)

```text
1. Patch everything          → apt update && apt upgrade
2. Non-root sudo user        → keys installed, sudo VERIFIED working
3. sshd lockdown             → keys only, no root login (test before disconnecting!)
4. Firewall                  → ufw default-deny, allow SSH FIRST, then enable
5. Brute-force response      → fail2ban with sshd jail
6. Automatic security patches → unattended-upgrades
7. Baseline audit            → record open ports + services while the box is clean
```

---

## USERS & PRIVILEGES

```bash
sudo adduser deploy                 # Create user (prompts for password)
sudo usermod -aG sudo deploy        # Grant sudo
groups deploy                       # Verify group membership
```

```bash
# From your LOCAL machine — install your key for the new user:
ssh-copy-id deploy@host

# On the server — VERIFY before locking anything down:
su - deploy
sudo whoami                         # Must print: root
```

```bash
sudo passwd -l root                 # Lock root's password (console/su blocked; sudo unaffected)
sudo visudo                         # ONLY way to edit sudoers (syntax-checked on save)
```

```text
⚠️ Order matters: confirm `ssh deploy@host` + sudo BOTH work in a fresh
terminal before disabling root login or password auth.
```

---

## SSHD LOCKDOWN (RECAP)

Full treatment in the [SSH Cheatsheet](ssh_cheatsheet.md) — the essentials:

```bash
# /etc/ssh/sshd_config
PasswordAuthentication no
PermitRootLogin no
ClientAliveInterval 60
ClientAliveCountMax 3
```

```bash
sudo sshd -t                        # Validate BEFORE restarting
sudo systemctl restart ssh          # Keep current session open; test with a NEW one
```

---

## UFW (FIREWALL)

### Setup (order prevents lockout)

```bash
sudo apt install ufw
sudo ufw default deny incoming      # Deny all inbound by default
sudo ufw default allow outgoing     # Allow all outbound
sudo ufw limit 22/tcp               # Allow SSH, rate-limited (6 conn / 30s per IP)
sudo ufw allow 80,443/tcp           # Web (if running nginx)
sudo ufw enable                     # ONLY after the SSH rule exists
sudo ufw status verbose             # Confirm
```

### Rule management

```bash
sudo ufw allow 8080/tcp                              # Open a port
sudo ufw allow from 203.0.113.0/24 to any port 5432  # Port open to one subnet only
sudo ufw allow in on tailscale0                      # Trust the entire tailnet interface
sudo ufw deny 8080/tcp                               # Explicit deny
sudo ufw status numbered                             # List rules with numbers
sudo ufw delete 3                                    # Delete by number
sudo ufw delete allow 8080/tcp                       # Delete by rule
sudo ufw app list                                    # Predefined profiles (OpenSSH, Nginx Full...)
sudo ufw logging on                                  # Log blocked packets → /var/log/ufw.log
sudo ufw reload
```

### Emergency

```bash
sudo ufw disable                    # Firewall off (rules preserved)
sudo ufw reset                      # Wipe ALL rules, start over
```

---

## FAIL2BAN

Watches logs, bans IPs that fail repeatedly. Zero config protects nothing — enable jails explicitly.

```bash
sudo apt install fail2ban
```

### Config — /etc/fail2ban/jail.local (NEVER edit jail.conf)

```bash
[DEFAULT]
bantime  = 1h
findtime = 10m                      # Window for counting failures
maxretry = 5
bantime.increment = true            # Repeat offenders get exponentially longer bans
ignoreip = 127.0.0.1/8 100.64.0.0/10   # Never ban yourself (localhost + tailnet)

[sshd]
enabled = true
# If this box logs SSH to the journal only (no /var/log/auth.log):
# backend = systemd

[nginx-http-auth]
enabled = true                      # Bans repeated basic-auth failures

[nginx-limit-req]
enabled = true                      # Bans IPs tripping limit_req (see nginx sheet)

[nginx-botsearch]
enabled = true                      # Bans scanners probing for /wp-admin etc.
```

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client reload         # Apply config changes
```

### Operations

```bash
sudo fail2ban-client status                     # List active jails
sudo fail2ban-client status sshd                # Banned IPs + failure counts
sudo fail2ban-client banned                     # All bans across all jails
sudo fail2ban-client set sshd unbanip 1.2.3.4   # Unban
sudo fail2ban-client set sshd banip 1.2.3.4     # Manual ban
sudo tail -f /var/log/fail2ban.log              # Watch bans happen live
```

---

## AUTOMATIC SECURITY UPDATES

```bash
sudo apt install unattended-upgrades apt-listchanges
sudo dpkg-reconfigure -plow unattended-upgrades   # Answer Yes → enables the timer
```

### /etc/apt/apt.conf.d/20auto-upgrades

```bash
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

### /etc/apt/apt.conf.d/50unattended-upgrades (key options)

```bash
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";        # Only if the box can reboot itself
Unattended-Upgrade::Automatic-Reboot-Time "03:30";
```

```bash
sudo unattended-upgrade --dry-run --debug   # Test what would happen
ls /var/log/unattended-upgrades/            # What it actually did
cat /var/run/reboot-required 2>/dev/null    # Kernel updated, reboot pending?
```

```text
💡 Default scope is security updates only — safe. Services running as
systemd units (Restart=always) come back after Automatic-Reboot; things
launched from a tmux session do NOT. Pair with tmux-resurrect or,
better, promote agents to systemd services (see systemd sheet).
```

---

## TAILNET-FIRST PATTERN (TAILSCALE)

The strongest hardening move: services listen on the tailnet (or loopback), the public internet sees only 80/443 — or nothing.

```bash
tailscale ip -4                     # This box's tailnet address (100.x.y.z)
tailscale status                    # Peers + connectivity
```

```bash
# Trust the tailnet interface wholesale:
sudo ufw allow in on tailscale0

# Bind services to the tailnet IP or loopback instead of 0.0.0.0:
#   - dashboards → 127.0.0.1, fronted by nginx allow/deny (see nginx sheet)
#   - internal APIs → 100.x.y.z (tailnet IP)
```

```bash
# Endgame — close public SSH entirely, reach it over the tailnet only:
sudo ufw delete limit 22/tcp
# ⚠️ ONLY after proving `ssh deploy@100.x.y.z` works from another tailnet
# device, and understanding: if tailscaled dies, so does your access.
```

```text
💡 Tailscale needs NO inbound firewall rules — it establishes outbound
connections (UDP 41641, HTTPS fallback), which default-allow-outgoing covers.
```

---

## AUDIT & MONITORING

### What's listening? (run while the box is clean, save the output)

```bash
sudo ss -tlnp                       # TCP listeners + owning process
sudo ss -ulnp                       # UDP listeners
sudo lsof -i -P -n | grep LISTEN    # Alternative view
```

### Who's been here?

```bash
last -n 20                          # Recent successful logins
sudo lastb -n 20                    # Recent FAILED logins (brute-force volume check)
w                                   # Who is logged in right now
sudo journalctl -u ssh -g "Accepted" --since today    # Successful SSH auths
```

### Key & persistence audit

```bash
# Every authorized key on the box — recognize ALL of them:
for f in /root/.ssh/authorized_keys /home/*/.ssh/authorized_keys; do
    echo "== $f"; sudo cat "$f" 2>/dev/null
done

# Scheduled-task persistence:
crontab -l
sudo ls /etc/cron.*/ /var/spool/cron/crontabs/ 2>/dev/null
systemctl list-timers

# SUID binaries (compare against your clean baseline):
sudo find / -perm -4000 -type f 2>/dev/null
```

### One-command audit

```bash
sudo apt install lynis
sudo lynis audit system             # Scored report + prioritized suggestions
```

---

## COMMON WORKFLOWS

### Fresh VPS, first hour

```bash
sudo apt update && sudo apt upgrade -y
sudo adduser deploy && sudo usermod -aG sudo deploy
# (from local) ssh-copy-id deploy@host → verify login + sudo
# sshd: keys only, no root (see recap above) → sshd -t → restart ssh
sudo ufw default deny incoming && sudo ufw default allow outgoing
sudo ufw limit 22/tcp && sudo ufw enable
sudo apt install fail2ban && sudo -e /etc/fail2ban/jail.local   # sshd jail
sudo systemctl enable --now fail2ban
sudo apt install unattended-upgrades && sudo dpkg-reconfigure -plow unattended-upgrades
sudo ss -tlnp > ~/baseline-ports.txt    # Snapshot the clean state
```

### Brute-force noise check

```bash
sudo lastb | awk '{print $3}' | sort | uniq -c | sort -rn | head   # Top attacking IPs
sudo fail2ban-client status sshd                                   # Is the jail catching them?
# High volume + public 22 open? → consider the tailnet-only endgame above
```

### Expose a new service safely

```bash
# 1. Bind it to 127.0.0.1 or the tailnet IP — NOT 0.0.0.0
# 2. Need public access? Front it with nginx (TLS + rate limit + allow/deny)
# 3. Only then: sudo ufw allow 443/tcp — never the service's raw port
```

---

## TIPS & GOTCHAS

- **`ufw enable` before an SSH rule = lockout.** The setup order above exists for a reason. On a fresh box, add the SSH rule first, always.
- **Docker punches through ufw.** `-p 8080:80` inserts iptables rules AHEAD of ufw — the port is public even when ufw says deny. Fix: publish to loopback (`-p 127.0.0.1:8080:80`) and front with nginx. Do not set `"iptables": false` in daemon.json — it breaks container networking.
- **`jail.local`, never `jail.conf`** — package upgrades overwrite jail.conf; your edits silently vanish on the next apt upgrade.
- **fail2ban is only as good as the log it reads** — an nginx jail pointed at a rotated/renamed logpath bans nobody. `fail2ban-client status <jail>` shows the file it's watching; zero "Total failed" forever is a smell.
- **Put your own networks in `ignoreip`** — fat-fingering your passphrase five times from your laptop shouldn't ban your tailnet.
- **`ufw allow` beats `ufw limit` for automation sources** — `limit` (6/30s) will throttle legitimate rapid connections from CI/scripts; use a source-scoped `allow from <ip>` for those and `limit` for the open internet.
- **Automatic-Reboot on a box running stateful work** — anything living only in tmux dies at 03:30. Promote it to a systemd unit with `Restart=always` first.
- **Audit against a baseline, not vibes** — `diff ~/baseline-ports.txt <(sudo ss -tlnp)` turns "does this look right?" into an actual answer.
- **Hardening is layered, not either/or** — ufw limits exposure, fail2ban punishes attempts, sshd config removes the password attack surface, tailnet binding removes the target. Each covers another's failure.

---
*Last Updated: 2026-07*

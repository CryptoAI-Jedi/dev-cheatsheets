# Linux (Debian/Ubuntu) Cheatsheet

> Debian-family administration and CLI troubleshooting: apt, systemd, networking, and the diagnostic loop (process → disk → network → logs). The baseline sheet for every VPS in the fleet.

---

## Table of Contents
- [System Info](#system-info)
- [Process & Performance](#process--performance)
- [CPU & Memory](#cpu--memory)
- [Disk & Filesystem](#disk--filesystem)
- [Networking](#networking)
- [Ports & Connections](#ports--connections)
- [Firewall](#firewall)
- [Logs](#logs)
- [Services (systemd)](#services-systemd)
- [File Permissions & Ownership](#file-permissions--ownership)
- [Search & Text](#search--text)
- [SSH](#ssh)
- [Package Management (apt)](#package-management-apt)
- [Tips & Gotchas](#tips--gotchas)

---

## SYSTEM INFO

```text
uname -a                           → Kernel & OS info
hostnamectl                        → Hostname, OS, kernel details
uptime                             → System uptime & load averages
whoami                             → Current user
id                                 → User ID, group memberships
lsb_release -a                     → Distro version (Debian/Ubuntu)
cat /etc/os-release                → OS details (universal)
```

---

## PROCESS & PERFORMANCE

```text
top                                → Live process monitor
htop                               → Better live monitor (if installed)
ps aux                             → All running processes
ps aux | grep process_name         → Find specific process
kill PID                           → Graceful kill (SIGTERM)
kill -9 PID                        → Force kill (SIGKILL — no cleanup)
pkill process_name                 → Kill by name
pgrep process_name                 → Find PID by name
nice -n 10 command                 → Run with lower priority
renice -n 5 -p PID                 → Change priority of running process
```

---

## CPU & MEMORY

```text
free -h                            → RAM usage (human-readable)
vmstat 1 5                         → CPU/mem stats (1s interval, 5x)
cat /proc/cpuinfo                  → CPU details
lscpu                              → CPU architecture summary
mpstat                             → Per-CPU stats (sysstat pkg)
```

---

## DISK & FILESYSTEM

```text
df -h                              → Disk space per filesystem
du -sh /path                       → Size of directory
du -sh * | sort -rh | head -10     → Top 10 largest items
lsblk                              → List block devices
blkid                              → Show UUIDs of block devices
fdisk -l                           → Partition table
mount                              → Show mounted filesystems
mount /dev/sdX /mnt                → Mount device
umount /mnt                        → Unmount
fsck /dev/sdX                      → Filesystem check (device must be unmounted)
```

---

## NETWORKING

```text
ip a                               → IP addresses (replaces ifconfig)
ip link show                       → Network interfaces status
ip route show                      → Routing table
ip route get 8.8.8.8               → Route used to reach host
ping -c 4 example.com              → ICMP connectivity test
traceroute example.com             → Hop-by-hop path
mtr example.com                    → Live traceroute (better)
curl -I https://example.com        → HTTP headers only
curl -v https://example.com        → Verbose HTTP request
wget -q --spider https://example.com   → Check URL reachability
dig example.com                    → Detailed DNS query
dig +short example.com             → Just the IP
host example.com                   → Simple DNS resolution
cat /etc/resolv.conf               → Current DNS servers
cat /etc/hosts                     → Local hostname overrides
```

---

## PORTS & CONNECTIONS

```text
ss -tulnp                          → Open ports & listening services
ss -s                              → Socket summary
lsof -i :8080                      → What's using port 8080
lsof -i tcp                        → All TCP connections
nc -zv host 22                     → Test if port is open
nmap -sV host                      → Port scan with service version
tcpdump -i eth0 port 80            → Capture traffic on port 80
```

---

## FIREWALL

```text
ufw status                         → Firewall status (Ubuntu)
ufw allow 22/tcp                   → Allow SSH
ufw deny 8080                      → Block port
ufw enable / ufw disable           → Toggle firewall
iptables -L -n -v                  → Raw iptables rules
```

Full default-deny setup, rate limiting, and fail2ban: [VPS Hardening sheet](vps_hardening_cheatsheet.md).

---

## LOGS

```text
journalctl -xe                     → Recent system logs (with context)
journalctl -u service_name         → Logs for specific service
journalctl -f                      → Follow live system logs
journalctl --since "1 hour ago"    → Logs from last hour
journalctl -k                      → Kernel ring buffer (modern dmesg)
tail -f /var/log/syslog            → Follow syslog
tail -f /var/log/auth.log          → SSH/auth events
grep "ERROR" /var/log/syslog       → Filter log for errors
dmesg                              → Kernel ring buffer (may need sudo)
```

Full filtering, priorities, and persistence: [systemd/journalctl sheet](systemd_journalctl_cheatsheet.md).

---

## SERVICES (systemd)

```text
systemctl status service_name      → Service status
systemctl start service_name       → Start service
systemctl stop service_name        → Stop service
systemctl restart service_name     → Restart service
systemctl enable service_name      → Enable on boot
systemctl disable service_name     → Disable on boot
systemctl enable --now service     → Enable + start in one step
systemctl list-units --type=service → List all services
```

---

## FILE PERMISSIONS & OWNERSHIP

```text
ls -la                             → List with permissions
chmod 755 file                     → rwxr-xr-x
chmod +x file                      → Make executable
chown user:group file              → Change owner
chown -R user:group /dir           → Recursive ownership change
umask                              → Default permission mask
```

---

## SEARCH & TEXT

```text
find /path -name "*.log"           → Find files by name
find /path -mtime -1               → Files modified in last 24hrs
find /path -size +100M             → Files larger than 100MB
grep -r "pattern" /path            → Recursive string search
grep -n "error" file.log           → Show line numbers
grep -i "error" file.log           → Case-insensitive search
awk '{print $1}' file              → Print first column
sed 's/old/new/g' file             → Replace string in output
cut -d: -f1 /etc/passwd            → Cut by delimiter, print field
sort file | uniq -c | sort -rn     → Count duplicates, sort by freq
```

Regex syntax across grep/sed/awk: [regex sheet](regex_cheatsheet.md).

---

## SSH

```text
ssh user@host                      → Connect to remote host
ssh -p 2222 user@host              → Custom port
ssh -i ~/.ssh/key.pem user@host    → Connect with key file
ssh-keygen -t ed25519              → Generate SSH key pair
ssh-copy-id user@host              → Copy public key to remote host
scp file.txt user@host:/path       → Secure copy to remote
scp user@host:/path file.txt       → Secure copy from remote
rsync -avz /src user@host:/dest    → Sync files (resumable)
```

Config, tunnels, ProxyJump: [SSH sheet](ssh_cheatsheet.md). Backups: [rsync sheet](rsync_backup_cheatsheet.md).

---

## PACKAGE MANAGEMENT (apt)

```text
apt update                         → Refresh package index
apt upgrade                        → Upgrade installed packages
apt full-upgrade                   → Upgrade + handle changed dependencies (dist-upgrade)
apt list --upgradable              → Preview pending upgrades
apt install package                → Install package
apt remove package                 → Remove package (keep config)
apt purge package                  → Remove package + config files
apt autoremove                     → Remove unused dependencies
apt search keyword                 → Search for package
apt show package                   → Package details
dpkg -l | grep package             → Check if installed
dpkg -L package                    → List files installed by package
dpkg -S /usr/bin/file              → Which package owns a file
```

Automatic security updates (unattended-upgrades): [VPS Hardening sheet](vps_hardening_cheatsheet.md).

---

## TIPS & GOTCHAS

- **`apt` is for humans, `apt-get` is for scripts** — apt's CLI warns its interface isn't stable; in cron jobs and automation use `apt-get -y` (and `DEBIAN_FRONTEND=noninteractive` for truly unattended runs).
- **`ifconfig`/`netstat` are legacy** — they need the net-tools package and are frozen; `ip` and `ss` are the maintained replacements and already installed. Learn the new ones; the old muscle memory dies eventually.
- **`kill -9` is the last resort, not the first** — SIGKILL skips cleanup handlers (temp files, locks, child processes). Plain `kill` (SIGTERM), wait a few seconds, then escalate.
- **Ubuntu double-logs** — the same events land in journald AND /var/log/syslog (rsyslog runs alongside). Pick journalctl as primary; the text files are the fallback/grep-friendly copy.
- **`fsck` on a mounted filesystem corrupts it** — unmount first, or run from recovery for the root fs. Modern journaling filesystems rarely need it anyway.
- **`du` and `df` disagreeing means deleted-but-open files** — a process holds a deleted log file open, df shows full, du can't see it. `lsof +L1` finds the holder; restart it to reclaim.
- **Reboot-required is a real state** — kernel/libc updates need it: `cat /var/run/reboot-required` says so, `checkrestart`/`needrestart` name the services running stale libraries.

---
*Last Updated: 2026-07*

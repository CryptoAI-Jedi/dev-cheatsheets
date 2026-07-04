# Linux (CentOS / RHEL) Cheatsheet

> Red Hat family administration: dnf, firewalld, SELinux, and LVM; the four things that make a RHEL box feel different from Debian. SELinux is a feature, not an obstacle; learn the four troubleshooting commands before disabling anything.

---

## Table of Contents
- [System Info](#system-info)
- [Package Management (dnf / rpm)](#package-management-dnf--rpm)
- [Firewall (firewalld)](#firewall-firewalld)
- [SELinux](#selinux)
- [Networking](#networking)
- [Process & Performance](#process--performance)
- [Disk & Filesystem (LVM)](#disk--filesystem-lvm)
- [Logs](#logs)
- [Services (systemd)](#services-systemd)
- [User & Group Management](#user--group-management)
- [SSH](#ssh)
- [Useful Tools](#useful-tools)
- [Tips & Gotchas](#tips--gotchas)

---

## SYSTEM INFO

```text
uname -a                           → Kernel & OS info
hostnamectl                        → Hostname, OS, kernel details
cat /etc/redhat-release            → RHEL/CentOS version
cat /etc/os-release                → Full OS details
uptime                             → System uptime & load averages
whoami / id                        → Current user & groups
lscpu                              → CPU architecture summary
cat /proc/meminfo                  → Detailed memory info
dmidecode -t system                → Hardware info (requires root)
```

---

## PACKAGE MANAGEMENT (dnf / rpm)

dnf is default on RHEL 8+ / CentOS Stream; yum (RHEL 7) accepts the same syntax.

```text
dnf upgrade                        → Update all packages (preferred term; update = alias)
dnf install nginx                  → Install package
dnf remove nginx                   → Remove package
dnf autoremove                     → Remove unused dependencies
dnf search keyword                 → Search for package
dnf info nginx                     → Package details
dnf list installed                 → List installed packages
dnf list installed | grep nginx    → Check if installed
dnf provides /usr/bin/python3      → Which package owns a file
dnf history                        → Transaction history
dnf history undo 5                 → UNDO transaction #5 (underrated superpower)
dnf clean all                      → Clear cache
dnf repolist                       → List enabled repos
dnf repolist all                   → List all repos
dnf config-manager --enable repo   → Enable a repo
dnf config-manager --disable repo  → Disable a repo
dnf install epel-release           → Extra Packages for Enterprise Linux (most 3rd-party tools)
```

### RPM (low-level)

```text
rpm -qa                            → List all installed RPMs
rpm -qi package                    → Package info
rpm -ql package                    → List files in package
rpm -qf /usr/bin/nginx             → Which package owns file
rpm -ivh package.rpm               → Install local RPM
rpm -e package                     → Remove RPM
```

---

## FIREWALL (firewalld)

```bash
systemctl enable --now firewalld
firewall-cmd --state               # Running / not running
firewall-cmd --get-default-zone    # Show default zone
firewall-cmd --get-active-zones    # Active zones + their interfaces
firewall-cmd --list-all            # All rules for default zone
firewall-cmd --list-all --zone=public
```

### Allow / block

```bash
firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --remove-service=http --permanent
firewall-cmd --remove-port=8080/tcp --permanent
firewall-cmd --reload              # Permanent rules apply ONLY after this
```

```bash
# Common services:
firewall-cmd --add-service=ssh
firewall-cmd --add-service=mysql
firewall-cmd --add-service=postgresql
```

### Rich rules (advanced)

```bash
firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.168.1.0/24 accept' --permanent
firewall-cmd --add-rich-rule='rule family=ipv4 source address=10.0.0.5 reject' --permanent
firewall-cmd --reload
```

---

## SELINUX

```bash
getenforce                         # Enforcing / Permissive / Disabled
sestatus                           # Full SELinux status
setenforce 0                       # Permissive (temporary — logs, doesn't block)
setenforce 1                       # Enforcing (temporary)
# Permanent: /etc/selinux/config → SELINUX=enforcing|permissive|disabled
```

### Contexts

```bash
ls -Z /var/www/html                # File SELinux context
ps -Z                              # Process SELinux context
chcon -t httpd_sys_content_t file  # Change file context (temporary)
restorecon -Rv /var/www/html       # Restore default context (the usual fix)
```

### Booleans

```bash
getsebool -a                       # List all SELinux booleans
getsebool httpd_can_network_connect
setsebool -P httpd_can_network_connect on   # -P = persistent
```

### Troubleshooting (in this order)

```bash
ausearch -m avc -ts recent         # Recent SELinux denials
sealert -a /var/log/audit/audit.log   # Human-readable explanations
restorecon -Rv /suspect/path       # Fix #1 cause: wrong context
audit2allow -a                     # LAST resort: generate allow rules from denials
```

---

## NETWORKING

```text
ip a                               → IP addresses
ip link show                       → Interface status
ip route show                      → Routing table
nmcli device status                → Network devices (NetworkManager)
nmcli connection show              → All connections
nmcli connection up eth0           → Bring up connection
nmcli connection down eth0         → Bring down connection
nmtui                              → Text UI for NetworkManager
```

### Static IP (NetworkManager)

```bash
nmcli connection modify eth0 ipv4.addresses 192.168.1.10/24
nmcli connection modify eth0 ipv4.gateway 192.168.1.1
nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection modify eth0 ipv4.method manual
nmcli connection up eth0
```

```text
Config locations:
/etc/sysconfig/network-scripts/           → Legacy (RHEL 7)
/etc/NetworkManager/system-connections/   → NM keyfiles (RHEL 8+)
```

```text
ping -c 4 example.com              → ICMP test
traceroute example.com             → Hop-by-hop path
ss -tulnp                          → Open ports & services
dig example.com                    → DNS query
curl -I https://example.com        → HTTP headers
```

---

## PROCESS & PERFORMANCE

```text
top / htop                         → Live process monitor
ps aux | grep process_name         → Find process
kill -9 PID                        → Force kill
free -h                            → RAM usage
vmstat 1 5                         → CPU/mem stats
iostat -x 1                        → Disk I/O stats (sysstat)
sar -u 1 5                         → CPU usage history (sysstat)
lsof -i :8080                      → What's using port 8080
```

---

## DISK & FILESYSTEM (LVM)

```text
df -h                              → Disk space per filesystem
du -sh /path                       → Directory size
lsblk                              → Block devices
fdisk -l                           → Partition table
parted -l                          → GPT partition info
mount / umount /mnt                → Mount/unmount
/etc/fstab                         → Persistent mounts
```

### LVM (default layout on RHEL installs)

```bash
pvs; vgs; lvs                      # Physical / volume-group / logical-volume info
lvextend -L +10G /dev/vg/lv        # Extend logical volume
xfs_growfs /mount/point            # Grow XFS after extend (RHEL default fs)
resize2fs /dev/vg/lv               # Grow ext4 after extend
lvextend -r -L +10G /dev/vg/lv     # Extend + grow filesystem in one step
```

---

## LOGS

```text
journalctl -xe                     → Recent system logs
journalctl -u service_name -f      → Follow service logs
journalctl -b                      → Logs since last boot
journalctl -p err                  → Errors only
/var/log/messages                  → General system log (RHEL/CentOS)
/var/log/secure                    → Auth/SSH log (vs auth.log on Debian)
/var/log/audit/audit.log           → SELinux & audit events
/var/log/dnf.log                   → Package install history
tail -f /var/log/messages          → Follow system log
```

---

## SERVICES (systemd)

```text
systemctl status service           → Service status
systemctl start | stop | restart service
systemctl enable | disable service
systemctl enable --now service     → Enable + start in one step
systemctl list-units --state=failed → Failed services
systemctl daemon-reload            → After editing unit files
```

Full treatment: systemd/journalctl sheet.

---

## USER & GROUP MANAGEMENT

```text
useradd -m -s /bin/bash alice      → Create user with home & shell
usermod -aG wheel alice            → Add to wheel (sudo) group
passwd alice                       → Set password
userdel -r alice                   → Delete user + home dir
groupadd mygroup                   → Create group
groups alice                       → Show user's groups
id alice                           → UID, GID, groups
visudo                             → Edit sudoers safely
/etc/sudoers.d/                    → Drop-in sudo rules
```

```text
💡 wheel = the sudo group on RHEL/CentOS.
/etc/sudoers line: %wheel ALL=(ALL) ALL
```

---

## SSH

```bash
ssh user@host                      # Connect
ssh -i ~/.ssh/key.pem user@host    # Connect with key
systemctl restart sshd             # Unit IS sshd here (ssh on Ubuntu)
ssh-keygen -t ed25519
ssh-copy-id user@host
# Config: /etc/ssh/sshd_config — full treatment: SSH + Keys sheets
```

---

## USEFUL TOOLS

```text
tar -czf archive.tar.gz /path      → Create compressed archive
tar -xzf archive.tar.gz            → Extract archive
find /path -name "*.log" -mtime -1 → Files modified in last 24hrs
grep -r "pattern" /path            → Recursive search
sed -i 's/old/new/g' file          → In-place replace
awk '{print $1}' file              → Print first column
watch -n 5 df -h                   → Repeat command every 5s
crontab -e / crontab -l            → Edit / list cron jobs
/etc/cron.d/                       → System cron drop-in dir
```

---

## TIPS & GOTCHAS

- **"It works with SELinux disabled" means the context is wrong, not that SELinux is broken** — `restorecon -Rv` on the app's paths fixes the overwhelming majority of denials. Permissive mode is the diagnostic (logs without blocking); disabled is surrender.
- **firewalld's two-layer trap** — rules without `--permanent` vanish on reload/reboot; rules with `--permanent` don't apply until `--reload`. Changing anything means BOTH: add with `--permanent`, then `--reload`.
- **`dnf history undo` is the rollback most distros wish they had** — bad update? `dnf history`, find the transaction, undo it. Check it before reaching for snapshots.
- **XFS grows but never shrinks** — RHEL's default filesystem has no shrink operation; sizing a logical volume too big is a backup-and-recreate to fix. Grow incrementally (`lvextend -r`).
- **The unit is `sshd`, the log is `/var/log/secure`** — coming from Debian: `systemctl restart ssh` fails, and auth events aren't in auth.log. Small differences, big confusion at 2am.
- **EPEL is where the tools live** — htop, fail2ban, and most of the familiar kit aren't in base repos; `dnf install epel-release` first on any new box.
- **CentOS Stream tracks AHEAD of RHEL** — it's a rolling preview, not the stable rebuild old CentOS was. For RHEL-identical stability, the free tiers of Rocky/Alma (or RHEL's own developer subscription) are the successors.

---
*Last Updated: 2026-07*

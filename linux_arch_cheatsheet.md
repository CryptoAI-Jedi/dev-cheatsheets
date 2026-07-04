# Linux (Arch) Cheatsheet

> Arch administration: pacman, the AUR, and the rolling-release discipline that keeps a system healthy. Rule one: never partial-upgrade. Rule two: the Arch Wiki already answered your question.

---

## Table of Contents
- [System Info](#system-info)
- [Package Management (pacman)](#package-management-pacman)
- [AUR](#aur)
- [Firewall](#firewall)
- [Networking](#networking)
- [Services (systemd)](#services-systemd)
- [Process & Performance](#process--performance)
- [Disk & Filesystem](#disk--filesystem)
- [Bootloader](#bootloader)
- [Logs](#logs)
- [User & Group Management](#user--group-management)
- [Kernel & Updates](#kernel--updates)
- [System Maintenance & Recovery](#system-maintenance--recovery)
- [SSH](#ssh)
- [Useful Tools](#useful-tools)
- [Tips & Gotchas](#tips--gotchas)

---

## SYSTEM INFO

```text
uname -a                           → Kernel & OS info
hostnamectl                        → Hostname & OS details
cat /etc/arch-release              → Confirm Arch Linux
cat /etc/os-release                → Full OS details
uptime                             → Uptime & load averages
lscpu                              → CPU info
free -h                            → RAM usage
fastfetch                          → System summary (neofetch's maintained successor)
```

---

## PACKAGE MANAGEMENT (pacman)

```text
pacman -Syu                        → Full system upgrade (ALWAYS before installing)
pacman -S package                  → Install package
pacman -S package1 package2        → Install multiple packages
pacman -R package                  → Remove package (keep deps)
pacman -Rs package                 → Remove package + unused deps
pacman -Rns package                → Remove + deps + config files
pacman -Ss keyword                 → Search for package
pacman -Si package                 → Package info (remote)
pacman -Qi package                 → Package info (installed)
pacman -Ql package                 → List files installed by package
pacman -Qo /usr/bin/nginx          → Which package owns file
pacman -Q                          → List all installed packages
pacman -Qe                         → Explicitly installed packages
pacman -Qdt                        → Orphan packages (unused deps)
pacman -Sc                         → Clear old package cache
pacman -Scc                        → Clear ALL package cache (kills downgrade option)
pacman -U package.pkg.tar.zst      → Install local package file
pacman -Fy                         → Sync file database
pacman -F filename                 → Which package provides file
```

```bash
pacman -Qtdq | pacman -Rns -                 # Remove ALL orphans in one pipe
paccache -r                                  # Keep last 3 cache versions (pacman-contrib)
pacman -U /var/cache/pacman/pkg/pkg-1.0-x86_64.pkg.tar.zst   # Downgrade from cache
```

### /etc/pacman.conf quality of life

```text
Color                              → Colored output
ParallelDownloads = 5              → Concurrent package downloads
VerbosePkgLists                    → Table view for upgrades
```

---

## AUR

### Install yay (AUR helper) — most common

```bash
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si
```

### Using yay (same syntax as pacman)

```text
yay -Syu                           → Update system + AUR packages
yay -S package                     → Install from AUR
yay -Ss keyword                    → Search AUR
yay -R package                     → Remove AUR package
yay -Qm                            → List AUR/foreign packages
```

### paru (alternative AUR helper)

```bash
paru -Syu
paru -S package
```

### Manual AUR install (without helper)

```bash
git clone https://aur.archlinux.org/package.git
cd package && less PKGBUILD && makepkg -si    # READ the PKGBUILD — see Gotchas
```

---

## FIREWALL

No firewall ships enabled on Arch — pick one:

### ufw (easiest)

```bash
pacman -S ufw
systemctl enable --now ufw
ufw default deny incoming && ufw default allow outgoing
ufw allow 22/tcp
ufw enable
ufw status                         # Full workflow: VPS Hardening sheet
```

### nftables (native modern option)

```bash
systemctl status nftables
nft list ruleset                   # Show current rules
# Config: /etc/nftables.conf
systemctl restart nftables
```

---

## NETWORKING

```text
ip a                               → IP addresses
ip link show                       → Interface status
ip route show                      → Routing table
ss -tulnp                          → Open ports
ping -c 4 archlinux.org
dig domain.com                     → DNS query
curl -I https://example.com        → HTTP headers
```

### NetworkManager (if installed)

```bash
nmcli device status
nmcli connection show
nmcli connection up connection-name
nmtui                              # Text UI
```

### systemd-networkd (minimal installs)

```bash
systemctl status systemd-networkd
# Config directory: /etc/systemd/network/
```

```ini
# /etc/systemd/network/20-wired.network
[Match]
Name=eth0

[Network]
Address=192.168.1.10/24
Gateway=192.168.1.1
DNS=8.8.8.8
```

```bash
systemctl restart systemd-networkd
```

### DNS (systemd-resolved)

```bash
systemctl status systemd-resolved
resolvectl status                  # DNS info
resolvectl query domain.com        # DNS lookup
ls -l /etc/resolv.conf             # Should symlink into systemd-resolved
```

---

## SERVICES (systemd)

```bash
systemctl status service
systemctl start service            # Also: stop | restart
systemctl enable service           # Also: disable
systemctl enable --now service     # Enable + start in one step
systemctl list-units --state=failed
systemctl daemon-reload
systemctl list-unit-files --type=service
# Full treatment: systemd/journalctl sheet
```

---

## PROCESS & PERFORMANCE

```text
top / htop                         → Live process monitor
ps aux | grep process_name         → Find process
kill -9 PID                        → Force kill
free -h                            → RAM usage
vmstat 1 5                         → CPU/mem stats
iostat -x 1                        → Disk I/O (requires sysstat package)
lsof -i :8080                      → What's using port 8080
```

---

## DISK & FILESYSTEM

```text
df -h                              → Disk usage
du -sh /path                       → Directory size
lsblk                              → Block devices
fdisk -l                           → Partition table
parted -l                          → GPT partition info
mount / umount /mnt                → Mount/unmount
/etc/fstab                         → Persistent mounts
```

### btrfs (common on Arch installs)

```text
btrfs filesystem show              → Filesystem info
btrfs filesystem usage /           → Usage details
btrfs subvolume list /             → List subvolumes
btrfs subvolume create /mnt/sub    → Create subvolume
btrfs scrub start /                → Check for errors
```

---

## BOOTLOADER

### GRUB

```bash
grub-mkconfig -o /boot/grub/grub.cfg    # Regenerate GRUB config
grub-install --target=x86_64-efi --efi-directory=/boot   # Install GRUB EFI
# Settings: /etc/default/grub
```

### systemd-boot (common on Arch)

```bash
bootctl install                    # Install systemd-boot
bootctl status                     # Boot status
bootctl update                     # Update bootloader
# Config: /boot/loader/loader.conf
# Entries: /boot/loader/entries/
```

---

## LOGS

```text
journalctl -xe                     → Recent system logs
journalctl -f                      → Follow live logs
journalctl -u service_name -f      → Follow service logs
journalctl -b                      → Logs since last boot
journalctl -b -1                   → Previous boot logs
journalctl -p err                  → Errors only
journalctl --disk-usage            → Journal size
journalctl --vacuum-time=7d        → Trim old logs
```

```bash
# Arch uses journald only — no /var/log/messages. Persistent logging:
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
```

---

## USER & GROUP MANAGEMENT

```bash
useradd -m -s /bin/bash alice      # Create user
usermod -aG wheel alice            # Add to wheel group (sudo)
passwd alice                       # Set password
userdel -r alice                   # Delete user + home
groupadd mygroup
id alice
visudo                             # Edit sudoers — uncomment: %wheel ALL=(ALL:ALL) ALL
```

---

## KERNEL & UPDATES

```bash
uname -r                           # Running kernel version
pacman -Q linux                    # Installed kernel version
pacman -Syu                        # Full system + kernel update
mkinitcpio -P                      # Regenerate initramfs (after kernel changes)
```

### LTS kernel alongside stable (the safety net)

```bash
pacman -S linux-lts linux-lts-headers
grub-mkconfig -o /boot/grub/grub.cfg    # GRUB picks it up; systemd-boot needs an entry
# A broken kernel update becomes a boot-menu selection instead of a rescue-USB day
```

---

## SYSTEM MAINTENANCE & RECOVERY

### Before big upgrades

```bash
# Check https://archlinux.org/news/ for manual-intervention notices
checkupdates                       # Preview pending updates (pacman-contrib)
```

### Mirrors — keep fast

```bash
pacman -S reflector
reflector --country US --latest 10 --sort rate --save /etc/pacman.d/mirrorlist
```

### Fix broken system after bad update (chroot from Arch ISO)

```bash
mount /dev/sdX1 /mnt               # (+ mount /boot / other partitions as laid out)
arch-chroot /mnt
pacman -Syu                        # Try update again
pacman -S package                  # Reinstall broken package
mkinitcpio -P                      # If the kernel/initramfs is the casualty
```

### pacman keyring issues ("invalid or corrupted package")

```bash
pacman -S archlinux-keyring        # Usually sufficient
pacman-key --init
pacman-key --populate archlinux
```

---

## SSH

```bash
ssh user@host
ssh -i ~/.ssh/key user@host
systemctl enable --now sshd
ssh-keygen -t ed25519
ssh-copy-id user@host
# Config: /etc/ssh/sshd_config — full treatment: SSH + Keys sheets
```

---

## USEFUL TOOLS

```text
tar -czf archive.tar.gz /path      → Compress
tar -xzf archive.tar.gz            → Extract
find /path -name "*.log"           → Find files
grep -r "pattern" /path            → Search
sed -i 's/old/new/g' file          → In-place replace
watch -n 5 df -h                   → Repeat command
crontab -e                         → Edit cron jobs (or systemd timers — see systemd sheet)
```

---

## TIPS & GOTCHAS

- **NEVER `pacman -Sy package`** — syncing the database without upgrading, then installing, is a partial upgrade: the new package links against libraries you don't have yet. It's the #1 way to break Arch. `-Syu` always; install after.
- **Handle your `.pacnew` files** — when you've edited a config, pacman saves the package's new version as `file.pacnew` instead of overwriting. Ignore them for a year and configs drift dangerously. `pacdiff` (pacman-contrib) walks you through merging.
- **Update the keyring first on stale systems** — a machine not updated for months fails with signature errors; `pacman -S archlinux-keyring` before `-Syu` clears it.
- **Read the PKGBUILD** — the AUR is user-submitted; the helper conveniently shows the build script *because you're supposed to read it*. It runs as you, and `makepkg` deps can pull anything.
- **AUR packages rebuild, they don't just update** — after big system upgrades (new python, new openssl), `yay -Syu` may need to rebuild AUR packages against the new libs; a segfaulting AUR tool usually just needs `yay -S --rebuild package`.
- **Keep linux-lts installed** — costs a few hundred MB, converts "system won't boot after kernel update" from an incident into a boot-menu choice.
- **Check Arch news before major updates** — manual-intervention notices are rare but real; `-Syu`-ing through one is how /boot ends up empty.
- **Don't `pacman -Scc` reflexively** — the cache is your offline downgrade path when a fresh package misbehaves. `paccache -r` keeps 3 versions and trims the rest.
- **Rolling means regularly** — updating weekly is routine; updating twice a year is an event. Long gaps compound keyring, dependency, and .pacnew debt simultaneously.

---
*Last Updated: 2026-07*

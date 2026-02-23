# linux_arch_cheatsheet

# SYSTEM INFO

---

uname -a                           → Kernel & OS info
hostnamectl                        → Hostname & OS details
cat /etc/arch-release              → Confirm Arch Linux
cat /etc/os-release                → Full OS details
uptime                             → Uptime & load averages
lscpu                              → CPU info
free -h                            → RAM usage
neofetch                           → System summary (if installed)

---

## PACKAGE MANAGEMENT (pacman)

pacman -Syu                        → Full system upgrade (ALWAYS run first)
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
pacman -Scc                        → Clear all package cache
pacman -U package.pkg.tar.zst      → Install local package file
pacman -Fy                         → Sync file database
pacman -F filename                 → Which package provides file

---

## AUR (Arch User Repository)

# Install yay (AUR helper) — most common

git clone [https://aur.archlinux.org/yay.git](https://aur.archlinux.org/yay.git)
cd yay && makepkg -si

# Using yay (same syntax as pacman)

yay -Syu                           → Update system + AUR packages
yay -S package                     → Install from AUR
yay -Ss keyword                    → Search AUR
yay -R package                     → Remove AUR package
yay -Qm                            → List AUR/foreign packages

# paru (alternative AUR helper)

paru -Syu
paru -S package

# Manual AUR install (without helper)

git clone [https://aur.archlinux.org/package.git](https://aur.archlinux.org/package.git)
cd package && makepkg -si

---

## FIREWALL (ufw or iptables — no default on Arch)

# Install and use ufw (easiest)

pacman -S ufw
systemctl enable --now ufw
ufw status
ufw allow 22/tcp
ufw allow http
ufw allow https
ufw deny 8080
ufw enable

# nftables (modern default on Arch)

systemctl status nftables
nft list ruleset                   → Show current rules
/etc/nftables.conf                 → Config file
systemctl restart nftables

---

## NETWORKING

ip a                               → IP addresses
ip link show                       → Interface status
ip route show                      → Routing table

# NetworkManager (if installed)

nmcli device status
nmcli connection show
nmcli connection up interface
nmtui                              → Text UI

# systemd-networkd (Arch default minimal install)

systemctl status systemd-networkd
/etc/systemd/network/              → Config directory

# Config example: /etc/systemd/network/20-wired.network

[Match]
Name=eth0

[Network]
Address=192.168.1.10/24
Gateway=192.168.1.1
DNS=8.8.8.8

systemctl restart systemd-networkd

# DNS (systemd-resolved)

systemctl status systemd-resolved
resolvectl status                  → DNS info
resolvectl query [domain.com](http://domain.com/)        → DNS lookup
/etc/resolv.conf                   → Should symlink to systemd-resolved

ping -c 4 [google.com](http://google.com/)
ss -tulnp                          → Open ports
dig [domain.com](http://domain.com/)                     → DNS query
curl -I [https://example.com](https://example.com/)        → HTTP headers

---

## SERVICES (systemd)

systemctl status service
systemctl start | stop | restart service
systemctl enable | disable service
systemctl enable --now service     → Enable + start in one step
systemctl list-units --state=failed
systemctl daemon-reload
systemctl list-unit-files --type=service

---

## PROCESS & PERFORMANCE

top / htop                         → Live process monitor
ps aux | grep process_name
kill -9 PID
free -h
vmstat 1 5
iostat -x 1                        → Requires sysstat package
lsof -i :8080

---

## DISK & FILESYSTEM

df -h                              → Disk usage
du -sh /path                       → Directory size
lsblk                              → Block devices
fdisk -l                           → Partition table
parted -l                          → GPT partition info
mount / umount /mnt
/etc/fstab                         → Persistent mounts

# btrfs (common on Arch installs)

btrfs filesystem show              → Filesystem info
btrfs filesystem usage /           → Usage details
btrfs subvolume list /             → List subvolumes
btrfs subvolume create /mnt/sub    → Create subvolume
btrfs scrub start /                → Check for errors

---

## BOOTLOADER

# GRUB (most common)

grub-mkconfig -o /boot/grub/grub.cfg   → Regenerate GRUB config
grub-install --target=x86_64-efi --efi-directory=/boot  → Install GRUB EFI
/etc/default/grub                       → GRUB settings

# systemd-boot (alternative, common on Arch)

bootctl install                         → Install systemd-boot
bootctl status                          → Boot status
bootctl update                          → Update bootloader
/boot/loader/loader.conf                → Bootloader config
/boot/loader/entries/                   → Boot entries

---

## LOGS

journalctl -xe                     → Recent system logs
journalctl -f                      → Follow live logs
journalctl -u service_name -f      → Follow service logs
journalctl -b                      → Logs since last boot
journalctl -b -1                   → Previous boot logs
journalctl -p err                  → Errors only
journalctl --disk-usage            → Journal size
journalctl --vacuum-time=7d        → Trim old logs

# Arch uses journald only — no /var/log/messages by default

# Enable persistent logging:

mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal

---

## USER & GROUP MANAGEMENT

useradd -m -s /bin/bash alice      → Create user
usermod -aG wheel alice            → Add to wheel group (sudo)
passwd alice                       → Set password
userdel -r alice                   → Delete user + home
groupadd mygroup
id alice
visudo                             → Edit sudoers

# Uncomment in sudoers: %wheel ALL=(ALL:ALL) ALL

---

## KERNEL & UPDATES

uname -r                           → Running kernel version
pacman -Q linux                    → Installed kernel version
pacman -S linux linux-headers      → Update kernel
mkinitcpio -P                      → Regenerate initramfs (after kernel changes)
pacman -Syu                        → Full system + kernel update

# Multiple kernels (e.g. LTS alongside stable)

pacman -S linux-lts linux-lts-headers

# Add boot entry for LTS kernel

---

## ARCH-SPECIFIC TIPS

# Rolling release — update regularly

pacman -Syu                        → Run before installing anything

# Arch Wiki — always check first

[https://wiki.archlinux.org](https://wiki.archlinux.org/)

# Mirrors — keep fast

pacman -S reflector
reflector --country US --latest 10 --sort rate --save /etc/pacman.d/mirrorlist

# Fix broken system after bad update

# Boot from Arch ISO, chroot:

mount /dev/sdX1 /mnt
arch-chroot /mnt
pacman -Syu                        → Try update again
pacman -S package                  → Reinstall broken package

# pacman keyring issues

pacman -S archlinux-keyring
pacman-key --init
pacman-key --populate archlinux

---

## SSH

ssh user@host
ssh -i ~/.ssh/key user@host
systemctl enable --now sshd
/etc/ssh/sshd_config               → SSH server config
ssh-keygen -t ed25519
ssh-copy-id user@host

---

## USEFUL TOOLS

tar -czf archive.tar.gz /path      → Compress
tar -xzf archive.tar.gz            → Extract
find /path -name "*.log"           → Find files
grep -r "pattern" /path            → Search
sed -i 's/old/new/g' file         → In-place replace
watch -n 5 df -h                   → Repeat command
crontab -e                         → Edit cron jobs
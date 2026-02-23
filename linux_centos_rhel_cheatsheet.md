# linux_centos_rhel_cheatsheet

# SYSTEM INFO

---

uname -a                           → Kernel & OS info
hostnamectl                        → Hostname, OS, kernel details
cat /etc/redhat-release            → RHEL/CentOS version
cat /etc/os-release                → Full OS details
uptime                             → System uptime & load averages
whoami / id                        → Current user & groups
lscpu                              → CPU architecture summary
cat /proc/meminfo                  → Detailed memory info
dmidecode -t system                → Hardware info (requires root)

---

## PACKAGE MANAGEMENT (dnf / yum)

# dnf is default on RHEL 8+ / CentOS Stream 8+

# yum is default on RHEL/CentOS 7

dnf update                         → Update all packages
dnf upgrade                        → Same as update (dnf preferred term)
dnf install nginx                  → Install package
dnf remove nginx                   → Remove package
dnf autoremove                     → Remove unused dependencies
dnf search keyword                 → Search for package
dnf info nginx                     → Package details
dnf list installed                 → List installed packages
dnf list installed | grep nginx    → Check if installed
dnf provides /usr/bin/python3      → Find which package owns a file
dnf history                        → Transaction history
dnf history undo 5                 → Undo transaction #5
dnf clean all                      → Clear cache
dnf repolist                       → List enabled repos
dnf repolist all                   → List all repos
dnf config-manager --enable repo   → Enable a repo
dnf config-manager --disable repo  → Disable a repo

# RPM (low-level)

rpm -qa                            → List all installed RPMs
rpm -qi package                    → Package info
rpm -ql package                    → List files in package
rpm -qf /usr/bin/nginx             → Which package owns file
rpm -ivh package.rpm               → Install local RPM
rpm -e package                     → Remove RPM

---

## FIREWALL (firewalld)

systemctl status firewalld         → Check firewall status
systemctl start firewalld          → Start firewall
systemctl enable firewalld         → Enable on boot

firewall-cmd --state               → Running / not running
firewall-cmd --get-default-zone    → Show default zone
firewall-cmd --get-active-zones    → Show active zones + interfaces
firewall-cmd --list-all            → All rules for default zone
firewall-cmd --list-all --zone=public

# Allow/block (--permanent makes it persist across reboots)

firewall-cmd --add-service=http --permanent
firewall-cmd --add-service=https --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --remove-service=http --permanent
firewall-cmd --remove-port=8080/tcp --permanent
firewall-cmd --reload              → Apply permanent rules

# Common services

firewall-cmd --add-service=ssh
firewall-cmd --add-service=mysql
firewall-cmd --add-service=postgresql

# Rich rules (advanced)

firewall-cmd --add-rich-rule='rule family=ipv4 source address=192.168.1.0/24 accept' --permanent
firewall-cmd --add-rich-rule='rule family=ipv4 source address=10.0.0.5 reject' --permanent

---

## SELINUX

getenforce                         → Enforcing / Permissive / Disabled
sestatus                           → Full SELinux status
setenforce 0                       → Set Permissive (temporary)
setenforce 1                       → Set Enforcing (temporary)

# Permanent change: /etc/selinux/config

SELINUX=enforcing | permissive | disabled

# Contexts

ls -Z /var/www/html                → Show file SELinux context
ps -Z                              → Show process SELinux context
chcon -t httpd_sys_content_t file  → Change file context (temp)
restorecon -Rv /var/www/html       → Restore default context

# Booleans

getsebool -a                       → List all SELinux booleans
getsebool httpd_can_network_connect
setsebool -P httpd_can_network_connect on   → Persistent boolean change

# Troubleshooting

ausearch -m avc -ts recent         → Recent SELinux denials
sealert -a /var/log/audit/audit.log → Human-readable SELinux alerts
audit2allow -a                     → Suggest allow rules from denials

---

## NETWORKING

ip a                               → IP addresses
ip link show                       → Interface status
ip route show                      → Routing table
nmcli device status                → Network devices (NetworkManager)
nmcli connection show              → All connections
nmcli connection up eth0           → Bring up connection
nmcli connection down eth0         → Bring down connection
nmcli device connect eth0          → Connect device
nmtui                              → Text UI for NetworkManager

# Static IP (NetworkManager)

nmcli connection modify eth0 ipv4.addresses 192.168.1.10/24
nmcli connection modify eth0 ipv4.gateway 192.168.1.1
nmcli connection modify eth0 ipv4.dns "8.8.8.8 8.8.4.4"
nmcli connection modify eth0 ipv4.method manual
nmcli connection up eth0

# Config files

/etc/sysconfig/network-scripts/    → Legacy config (RHEL 7)
/etc/NetworkManager/system-connections/  → NM connection files (RHEL 8+)

ping -c 4 [google.com](http://google.com/)               → ICMP test
traceroute [google.com](http://google.com/)              → Hop-by-hop path
ss -tulnp                          → Open ports & services
nslookup [domain.com](http://domain.com/)                → DNS lookup
dig [domain.com](http://domain.com/)                     → Detailed DNS query
curl -I [https://example.com](https://example.com/)        → HTTP headers

---

## PROCESS & PERFORMANCE

top / htop                         → Live process monitor
ps aux | grep process_name         → Find process
kill -9 PID                        → Force kill
free -h                            → RAM usage
vmstat 1 5                         → CPU/mem stats
iostat -x 1                        → Disk I/O stats
sar -u 1 5                         → CPU usage history (sysstat)
lsof -i :8080                      → What's using port 8080

---

## DISK & FILESYSTEM

df -h                              → Disk space per filesystem
du -sh /path                       → Directory size
lsblk                              → Block devices
fdisk -l                           → Partition table
parted -l                          → GPT partition info
pvs / vgs / lvs                    → LVM physical/volume/logical info
lvextend -L +10G /dev/vg/lv        → Extend logical volume
xfs_growfs /mount/point            → Grow XFS after extend
resize2fs /dev/vg/lv               → Grow ext4 after extend
mount / umount /mnt                → Mount/unmount
/etc/fstab                         → Persistent mounts

---

## LOGS

journalctl -xe                     → Recent system logs
journalctl -u service_name -f      → Follow service logs
journalctl -b                      → Logs since last boot
journalctl -p err                  → Errors only
/var/log/messages                  → General system log (RHEL/CentOS)
/var/log/secure                    → Auth/SSH log (vs auth.log on Debian)
/var/log/audit/audit.log           → SELinux & audit events
/var/log/dnf.log                   → Package install history
tail -f /var/log/messages          → Follow system log

---

## SERVICES (systemd)

systemctl status service           → Service status
systemctl start | stop | restart service
systemctl enable | disable service
systemctl enable --now service     → Enable + start in one step
systemctl list-units --state=failed → Failed services
systemctl daemon-reload            → After editing unit files

---

## USER & GROUP MANAGEMENT

useradd -m -s /bin/bash alice      → Create user with home & shell
usermod -aG wheel alice            → Add to wheel (sudo) group
passwd alice                       → Set password
userdel -r alice                   → Delete user + home dir
groupadd mygroup                   → Create group
groups alice                       → Show user's groups
id alice                           → UID, GID, groups
visudo                             → Edit sudoers file safely
cat /etc/sudoers.d/alice           → User-specific sudo rules

# wheel group = sudo equivalent on RHEL/CentOS

# /etc/sudoers: %wheel ALL=(ALL) ALL

---

## SSH

ssh user@host                      → Connect
ssh -i ~/.ssh/key.pem user@host    → Connect with key
sshd_config location: /etc/ssh/sshd_config
systemctl restart sshd             → Restart SSH daemon
ssh-keygen -t ed25519              → Generate key pair
ssh-copy-id user@host              → Copy key to remote host

---

## USEFUL TOOLS

tar -czf archive.tar.gz /path      → Create compressed archive
tar -xzf archive.tar.gz            → Extract archive
find /path -name "*.log" -mtime -1 → Files modified in last 24hrs
grep -r "pattern" /path            → Recursive search
sed -i 's/old/new/g' file          → In-place replace
awk '{print $1}' file              → Print first column
watch -n 5 df -h                   → Repeat command every 5s
crontab -e                         → Edit user cron jobs
crontab -l                         → List cron jobs
/etc/cron.d/                       → System cron drop-in dir
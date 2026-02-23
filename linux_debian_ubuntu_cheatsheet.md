# linux_cli_troubleshooting_cheatsheet

# SYSTEM INFO

---

uname -a                           → Kernel & OS info
hostnamectl                        → Hostname, OS, kernel details
uptime                             → System uptime & load averages
whoami                             → Current user
id                                 → User ID, group memberships
lsb_release -a                     → Distro version (Debian/Ubuntu)
cat /etc/os-release                → OS details (universal)

---

## PROCESS & PERFORMANCE

top                                → Live process monitor
htop                               → Better live monitor (if installed)
ps aux                             → All running processes
ps aux | grep process_name         → Find specific process
kill PID                           → Graceful kill
kill -9 PID                        → Force kill
pkill process_name                 → Kill by name
pgrep process_name                 → Find PID by name
nice -n 10 command                 → Run with lower priority
renice -n 5 -p PID                 → Change priority of running process

---

## CPU & MEMORY

free -h                            → RAM usage (human-readable)
vmstat 1 5                         → CPU/mem stats (1s interval, 5x)
cat /proc/cpuinfo                  → CPU details
lscpu                              → CPU architecture summary
mpstat                             → Per-CPU stats (sysstat pkg)

---

## DISK & FILESYSTEM

df -h                              → Disk space per filesystem
du -sh /path                       → Size of directory
du -sh * | sort -rh | head -10     → Top 10 largest items
lsblk                              → List block devices
blkid                              → Show UUIDs of block devices
fdisk -l                           → Partition table
mount                              → Show mounted filesystems
mount /dev/sdX /mnt                → Mount device
umount /mnt                        → Unmount
fsck /dev/sdX                      → Filesystem check (unmounted)

---

## NETWORKING

ip a                               → IP addresses (replaces ifconfig)
ip link show                       → Network interfaces status
ip route show                      → Routing table
ip route get 8.8.8.8               → Route used to reach host
ping -c 4 [google.com](http://google.com/)               → ICMP connectivity test
traceroute [google.com](http://google.com/)              → Hop-by-hop path
mtr [google.com](http://google.com/)                     → Live traceroute (better)
curl -I [https://example.com](https://example.com/)        → HTTP headers only
curl -v [https://example.com](https://example.com/)        → Verbose HTTP request
wget -q --spider [https://example.com](https://example.com/)  → Check URL reachability
nslookup [domain.com](http://domain.com/)                → DNS lookup
dig [domain.com](http://domain.com/)                     → Detailed DNS query
dig +short [domain.com](http://domain.com/)              → Just the IP
host [domain.com](http://domain.com/)                    → Simple DNS resolution
cat /etc/resolv.conf               → Current DNS servers
cat /etc/hosts                     → Local hostname overrides

---

## PORTS & CONNECTIONS

ss -tulnp                          → Open ports & listening services
ss -s                              → Socket summary
netstat -tulnp                     → Same (older, may need install)
lsof -i :8080                      → What's using port 8080
lsof -i tcp                        → All TCP connections
nc -zv host 22                     → Test if port is open
nmap -sV host                      → Port scan with service version
tcpdump -i eth0 port 80            → Capture traffic on port 80

---

## FIREWALL

ufw status                         → Firewall status (Ubuntu)
ufw allow 22/tcp                   → Allow SSH
ufw deny 8080                      → Block port
ufw enable / ufw disable           → Toggle firewall
iptables -L -n -v                  → Raw iptables rules

---

## LOGS

journalctl -xe                     → Recent system logs (with context)
journalctl -u service_name         → Logs for specific service
journalctl -f                      → Follow live system logs
journalctl --since "1 hour ago"    → Logs from last hour
tail -f /var/log/syslog            → Follow syslog
tail -f /var/log/auth.log          → SSH/auth events
grep "ERROR" /var/log/syslog       → Filter log for errors
cat /var/log/dmesg                 → Kernel ring buffer

---

## SERVICES (systemd)

systemctl status service_name      → Service status
systemctl start service_name       → Start service
systemctl stop service_name        → Stop service
systemctl restart service_name     → Restart service
systemctl enable service_name      → Enable on boot
systemctl disable service_name     → Disable on boot
systemctl list-units --type=service → List all services

---

## FILE PERMISSIONS & OWNERSHIP

ls -la                             → List with permissions
chmod 755 file                     → rwxr-xr-x
chmod +x file                      → Make executable
chown user:group file              → Change owner
chown -R user:group /dir           → Recursive ownership change
umask                              → Default permission mask

---

## SEARCH & TEXT

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

---

## SSH

ssh user@host                      → Connect to remote host
ssh -p 2222 user@host              → Custom port
ssh -i ~/.ssh/key.pem user@host    → Connect with key file
ssh-keygen -t ed25519              → Generate SSH key pair
ssh-copy-id user@host              → Copy public key to remote host
scp file.txt user@host:/path       → Secure copy to remote
scp user@host:/path file.txt       → Secure copy from remote
rsync -avz /src user@host:/dest    → Sync files (resumable)

---

## PACKAGE MANAGEMENT (Debian/Ubuntu)

apt update                         → Refresh package index
apt upgrade                        → Upgrade installed packages
apt install package                → Install package
apt remove package                 → Remove package
apt autoremove                     → Remove unused dependencies
apt search keyword                 → Search for package
dpkg -l | grep package             → Check if installed
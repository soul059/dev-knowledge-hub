# Linux Security Hardening

A comprehensive guide to securing Linux systems through configuration best practices, access controls, and defensive measures.

## Table of Contents

1. [Security Fundamentals](#1-security-fundamentals)
2. [User and Access Security](#2-user-and-access-security)
3. [SSH Hardening](#3-ssh-hardening)
4. [Firewall Configuration](#4-firewall-configuration)
5. [File System Security](#5-file-system-security)
6. [Kernel Hardening](#6-kernel-hardening)
7. [Service Hardening](#7-service-hardening)
8. [Audit and Monitoring](#8-audit-and-monitoring)
9. [Intrusion Detection](#9-intrusion-detection)
10. [Security Automation](#10-security-automation)

---

## 1. Security Fundamentals

### Defense in Depth

Security should be implemented in layers:

```
┌─────────────────────────────────────┐
│         Physical Security           │
├─────────────────────────────────────┤
│         Network Security            │
├─────────────────────────────────────┤
│           Host Security             │
├─────────────────────────────────────┤
│       Application Security          │
├─────────────────────────────────────┤
│          Data Security              │
└─────────────────────────────────────┘
```

### Principle of Least Privilege

- Users get minimum permissions needed
- Services run with restricted privileges
- Root access is limited and audited

### Security Baseline Checklist

```bash
# Quick security audit
#!/bin/bash

echo "=== Security Quick Audit ==="

# Check for empty passwords
echo "[*] Users with empty passwords:"
sudo awk -F: '($2 == "") {print $1}' /etc/shadow

# Check for UID 0 accounts (root equivalents)
echo "[*] Accounts with UID 0:"
awk -F: '($3 == 0) {print $1}' /etc/passwd

# Check SSH root login
echo "[*] SSH root login status:"
grep "^PermitRootLogin" /etc/ssh/sshd_config

# Check running services
echo "[*] Listening services:"
ss -tulpn | grep LISTEN

# Check for SUID files
echo "[*] SUID files (first 10):"
find / -perm -4000 -type f 2>/dev/null | head -10
```

---

## 2. User and Access Security

### Password Policies

#### Configure Password Aging
```bash
# /etc/login.defs
PASS_MAX_DAYS   90      # Maximum password age
PASS_MIN_DAYS   7       # Minimum days between changes
PASS_MIN_LEN    12      # Minimum password length
PASS_WARN_AGE   14      # Warning days before expiry

# Apply to existing user
sudo chage -M 90 -m 7 -W 14 username
```

#### Password Complexity with PAM
```bash
# /etc/security/pwquality.conf
minlen = 14
dcredit = -1    # At least 1 digit
ucredit = -1    # At least 1 uppercase
lcredit = -1    # At least 1 lowercase
ocredit = -1    # At least 1 special char
maxrepeat = 3   # Max consecutive same chars
```

### Sudo Configuration

```bash
# /etc/sudoers (use visudo to edit)

# Require password for sudo
Defaults        timestamp_timeout=5
Defaults        passwd_tries=3

# Log all sudo commands
Defaults        logfile="/var/log/sudo.log"
Defaults        log_input, log_output

# Restrict specific users
username ALL=(ALL) /usr/bin/systemctl restart nginx
```

### Account Lockout

```bash
# /etc/pam.d/common-auth (Debian/Ubuntu)
# Add at the top:
auth required pam_tally2.so deny=5 unlock_time=900 onerr=fail

# Check failed attempts
sudo pam_tally2 --user=username

# Reset counter
sudo pam_tally2 --user=username --reset
```

### Disable Unused Accounts

```bash
# Lock an account
sudo usermod -L username

# Disable shell access
sudo usermod -s /usr/sbin/nologin username

# Set account expiration
sudo usermod -e 2024-12-31 username

# Find inactive accounts
sudo lastlog | grep "Never logged in"
```

---

## 3. SSH Hardening

### Secure SSH Configuration

```bash
# /etc/ssh/sshd_config

# Disable root login
PermitRootLogin no

# Use SSH Protocol 2 only
Protocol 2

# Disable password authentication (use keys)
PasswordAuthentication no
PubkeyAuthentication yes

# Limit users
AllowUsers admin deploy
AllowGroups sshusers

# Disable empty passwords
PermitEmptyPasswords no

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Restrict SSH to specific interface
ListenAddress 192.168.1.10

# Change default port (security through obscurity)
Port 2222

# Disable X11 forwarding
X11Forwarding no

# Disable TCP forwarding if not needed
AllowTcpForwarding no

# Use strong ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group16-sha512

# Limit authentication attempts
MaxAuthTries 3
MaxSessions 2
```

### SSH Key Management

```bash
# Generate strong key
ssh-keygen -t ed25519 -a 100 -C "user@hostname"

# Or RSA with 4096 bits
ssh-keygen -t rsa -b 4096 -C "user@hostname"

# Copy key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Set proper permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
```

### Fail2Ban for SSH

```bash
# Install fail2ban
sudo apt install fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

# Start service
sudo systemctl enable --now fail2ban

# Check status
sudo fail2ban-client status sshd
```

---

## 4. Firewall Configuration

### UFW (Uncomplicated Firewall)

```bash
# Enable UFW
sudo ufw enable

# Default policies
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow specific services
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow from specific IP
sudo ufw allow from 192.168.1.100

# Allow subnet
sudo ufw allow from 192.168.1.0/24 to any port 22

# Rate limiting
sudo ufw limit ssh

# Check status
sudo ufw status verbose
```

### iptables

```bash
# Flush existing rules
sudo iptables -F
sudo iptables -X

# Default policies
sudo iptables -P INPUT DROP
sudo iptables -P FORWARD DROP
sudo iptables -P OUTPUT ACCEPT

# Allow loopback
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A OUTPUT -o lo -j ACCEPT

# Allow established connections
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (with rate limiting)
sudo iptables -A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -m limit --limit 3/min --limit-burst 3 -j ACCEPT

# Allow HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Log dropped packets
sudo iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: "

# Save rules
sudo iptables-save > /etc/iptables/rules.v4
```

### nftables (Modern Replacement)

```bash
# /etc/nftables.conf
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;
        
        # Allow loopback
        iif lo accept
        
        # Allow established
        ct state established,related accept
        
        # Allow SSH
        tcp dport 22 ct state new limit rate 3/minute accept
        
        # Allow HTTP/HTTPS
        tcp dport { 80, 443 } accept
        
        # Log and drop
        log prefix "nftables dropped: " drop
    }
    
    chain forward {
        type filter hook forward priority 0; policy drop;
    }
    
    chain output {
        type filter hook output priority 0; policy accept;
    }
}
```

---

## 5. File System Security

### File Permissions Best Practices

```bash
# Critical file permissions
chmod 644 /etc/passwd
chmod 600 /etc/shadow
chmod 644 /etc/group
chmod 600 /etc/gshadow
chmod 700 /root
chmod 600 /boot/grub/grub.cfg
```

### Find and Fix Insecure Files

```bash
# Find world-writable files
find / -type f -perm -0002 -ls 2>/dev/null

# Find world-writable directories
find / -type d -perm -0002 -ls 2>/dev/null

# Find SUID/SGID files
find / -type f \( -perm -4000 -o -perm -2000 \) -ls 2>/dev/null

# Find files without owner
find / -nouser -o -nogroup 2>/dev/null
```

### Immutable Files

```bash
# Make file immutable (cannot be modified/deleted)
sudo chattr +i /etc/passwd

# Make file append-only
sudo chattr +a /var/log/secure

# View attributes
lsattr /etc/passwd

# Remove immutable flag
sudo chattr -i /etc/passwd
```

### Mount Options

```bash
# /etc/fstab security options

# /tmp with noexec, nosuid, nodev
tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0

# /var with nosuid
/dev/sda3 /var ext4 defaults,nosuid 0 2

# /home with nodev
/dev/sda4 /home ext4 defaults,nodev 0 2

# Options explained:
# noexec  - Prevent execution of binaries
# nosuid  - Ignore SUID/SGID bits
# nodev   - Ignore device files
```

### Access Control Lists (ACLs)

```bash
# Set ACL
setfacl -m u:username:rx /path/to/file
setfacl -m g:groupname:r /path/to/file

# View ACL
getfacl /path/to/file

# Remove ACL
setfacl -x u:username /path/to/file

# Recursive ACL
setfacl -R -m u:username:rx /path/to/directory

# Default ACL for new files
setfacl -d -m u:username:rx /path/to/directory
```

---

## 6. Kernel Hardening

### Sysctl Security Parameters

```bash
# /etc/sysctl.d/99-security.conf

# IP Spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2

# Log Martians (impossible addresses)
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0

# Disable IPv6 if not needed
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1

# Restrict core dumps
fs.suid_dumpable = 0

# Randomize memory space (ASLR)
kernel.randomize_va_space = 2

# Restrict kernel pointer exposure
kernel.kptr_restrict = 2

# Restrict dmesg access
kernel.dmesg_restrict = 1

# Apply changes
sudo sysctl -p /etc/sysctl.d/99-security.conf
```

### Disable Unnecessary Modules

```bash
# /etc/modprobe.d/blacklist.conf

# Disable USB storage
install usb-storage /bin/true

# Disable uncommon filesystems
install cramfs /bin/true
install freevxfs /bin/true
install jffs2 /bin/true
install hfs /bin/true
install hfsplus /bin/true
install squashfs /bin/true
install udf /bin/true

# Disable uncommon network protocols
install dccp /bin/true
install sctp /bin/true
install rds /bin/true
install tipc /bin/true
```

---

## 7. Service Hardening

### Disable Unnecessary Services

```bash
# List all services
systemctl list-unit-files --type=service

# Check enabled services
systemctl list-unit-files --state=enabled

# Disable unnecessary services
sudo systemctl disable --now cups      # Printing
sudo systemctl disable --now avahi-daemon  # mDNS
sudo systemctl disable --now bluetooth # Bluetooth
sudo systemctl disable --now postfix   # Mail server

# Mask services (prevent starting)
sudo systemctl mask ctrl-alt-del.target
```

### Systemd Service Hardening

```bash
# /etc/systemd/system/myapp.service
[Unit]
Description=My Secure Application
After=network.target

[Service]
Type=simple
User=appuser
Group=appgroup
ExecStart=/usr/bin/myapp

# Security options
NoNewPrivileges=yes
PrivateTmp=yes
PrivateDevices=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/lib/myapp
ProtectKernelTunables=yes
ProtectKernelModules=yes
ProtectControlGroups=yes
RestrictAddressFamilies=AF_INET AF_INET6 AF_UNIX
RestrictNamespaces=yes
RestrictRealtime=yes
RestrictSUIDSGID=yes
MemoryDenyWriteExecute=yes
LockPersonality=yes

# Resource limits
MemoryMax=512M
CPUQuota=50%

[Install]
WantedBy=multi-user.target
```

### Analyze Service Security

```bash
# Check service security level
systemd-analyze security

# Check specific service
systemd-analyze security nginx.service
```

---

## 8. Audit and Monitoring

### Auditd Configuration

```bash
# Install auditd
sudo apt install auditd

# /etc/audit/rules.d/audit.rules

# Delete all previous rules
-D

# Buffer size
-b 8192

# Failure mode (1=printk, 2=panic)
-f 1

# Monitor authentication
-w /etc/pam.d/ -p wa -k pam_changes
-w /etc/login.defs -p wa -k login_changes

# Monitor user/group changes
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/gshadow -p wa -k identity

# Monitor sudoers
-w /etc/sudoers -p wa -k sudoers
-w /etc/sudoers.d/ -p wa -k sudoers

# Monitor SSH
-w /etc/ssh/sshd_config -p wa -k sshd_config

# Monitor cron
-w /etc/crontab -p wa -k cron
-w /etc/cron.d/ -p wa -k cron
-w /var/spool/cron/ -p wa -k cron

# Monitor network config
-w /etc/hosts -p wa -k hosts
-w /etc/network/ -p wa -k network

# System calls - file deletion
-a always,exit -F arch=b64 -S unlink -S unlinkat -S rename -S renameat -F auid>=1000 -F auid!=4294967295 -k delete

# System calls - module loading
-w /sbin/insmod -p x -k modules
-w /sbin/rmmod -p x -k modules
-w /sbin/modprobe -p x -k modules
```

```bash
# Restart auditd
sudo systemctl restart auditd

# Search audit logs
sudo ausearch -k identity -ts today
sudo ausearch -ua root -ts recent

# Generate reports
sudo aureport --summary
sudo aureport --auth
sudo aureport --login
```

### Log Management

```bash
# Important log files
/var/log/auth.log      # Authentication logs
/var/log/syslog        # System logs
/var/log/kern.log      # Kernel logs
/var/log/faillog       # Failed login attempts
/var/log/lastlog       # Last login info
/var/log/wtmp          # Login records
/var/log/btmp          # Bad login attempts

# Monitor logs in real-time
sudo tail -f /var/log/auth.log

# Search for failed SSH attempts
grep "Failed password" /var/log/auth.log

# Check last logins
last -n 20
lastb -n 20  # Failed logins
```

### Centralized Logging with rsyslog

```bash
# /etc/rsyslog.d/50-remote.conf

# Send logs to remote server
*.* @@logserver.example.com:514

# Or use TLS
$DefaultNetstreamDriverCAFile /etc/ssl/certs/ca.pem
$ActionSendStreamDriver gtls
$ActionSendStreamDriverMode 1
$ActionSendStreamDriverAuthMode x509/name
*.* @@logserver.example.com:6514
```

---

## 9. Intrusion Detection

### AIDE (Advanced Intrusion Detection Environment)

```bash
# Install AIDE
sudo apt install aide

# Initialize database
sudo aideinit

# Move database
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Check for changes
sudo aide --check

# Update database after legitimate changes
sudo aide --update
sudo mv /var/lib/aide/aide.db.new /var/lib/aide/aide.db
```

```bash
# /etc/aide/aide.conf custom rules
# Monitor critical directories
/etc CONTENT_EX
/bin CONTENT_EX
/sbin CONTENT_EX
/usr/bin CONTENT_EX
/usr/sbin CONTENT_EX

# Exclude directories
!/var/log
!/var/cache
!/tmp
```

### Rootkit Detection

```bash
# Install rkhunter
sudo apt install rkhunter

# Update database
sudo rkhunter --update

# Run scan
sudo rkhunter --check --sk

# Install chkrootkit
sudo apt install chkrootkit

# Run scan
sudo chkrootkit
```

### ClamAV Antivirus

```bash
# Install ClamAV
sudo apt install clamav clamav-daemon

# Update virus database
sudo freshclam

# Scan directory
clamscan -r /home

# Scan and remove infected
clamscan -r --remove /home

# Scan with detailed output
clamscan -r -i /home  # Show only infected
```

---

## 10. Security Automation

### Automated Security Updates

```bash
# Install unattended-upgrades
sudo apt install unattended-upgrades

# Configure
sudo dpkg-reconfigure unattended-upgrades

# /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};

Unattended-Upgrade::Package-Blacklist {
    // "linux-image*";
};

Unattended-Upgrade::AutoFixInterruptedDpkg "true";
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
```

### Security Scanning Script

```bash
#!/bin/bash
# security-scan.sh - Automated security check

LOG_FILE="/var/log/security-scan.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "=== Security Scan: $DATE ===" >> $LOG_FILE

# Check for listening ports
echo "[Listening Ports]" >> $LOG_FILE
ss -tulpn | grep LISTEN >> $LOG_FILE

# Check for failed logins
echo "[Failed Logins (last 24h)]" >> $LOG_FILE
grep "Failed password" /var/log/auth.log | tail -20 >> $LOG_FILE

# Check for root logins
echo "[Root Logins]" >> $LOG_FILE
grep "session opened for user root" /var/log/auth.log | tail -10 >> $LOG_FILE

# Check disk usage
echo "[Disk Usage]" >> $LOG_FILE
df -h | grep -E '^/dev' >> $LOG_FILE

# Check for unauthorized SUID files
echo "[New SUID Files]" >> $LOG_FILE
find /usr /bin /sbin -perm -4000 -type f -mtime -1 2>/dev/null >> $LOG_FILE

# Check running processes
echo "[Processes by Root]" >> $LOG_FILE
ps aux | grep ^root | wc -l >> $LOG_FILE

echo "=== Scan Complete ===" >> $LOG_FILE
echo "" >> $LOG_FILE

# Send email alert if issues found
# mail -s "Security Scan Report" admin@example.com < $LOG_FILE
```

### Cron Jobs for Security

```bash
# /etc/cron.d/security-tasks

# Daily security scan
0 2 * * * root /usr/local/bin/security-scan.sh

# Weekly rootkit check
0 3 * * 0 root /usr/bin/rkhunter --check --sk --report-warnings-only

# Daily AIDE check
0 4 * * * root /usr/bin/aide --check | mail -s "AIDE Report" admin@example.com

# Update ClamAV daily
0 1 * * * root /usr/bin/freshclam --quiet
```

---

## Quick Reference: Hardening Checklist

```
☐ System Updates
  ☐ Enable automatic security updates
  ☐ Remove unnecessary packages

☐ User Security
  ☐ Disable root login
  ☐ Configure password policies
  ☐ Implement sudo restrictions
  ☐ Remove unused accounts

☐ SSH Security
  ☐ Disable password authentication
  ☐ Use SSH keys only
  ☐ Change default port
  ☐ Configure fail2ban

☐ Network Security
  ☐ Configure firewall (deny by default)
  ☐ Disable unused services
  ☐ Enable TCP SYN cookies

☐ File System
  ☐ Set proper permissions
  ☐ Configure mount options
  ☐ Implement file integrity monitoring

☐ Monitoring
  ☐ Configure auditd
  ☐ Set up log management
  ☐ Install intrusion detection

☐ Kernel
  ☐ Apply sysctl hardening
  ☐ Disable unused modules
```

---

## Resources

- [CIS Benchmarks](https://www.cisecurity.org/cis-benchmarks/) - Industry security standards
- [NIST Guidelines](https://www.nist.gov/) - Security frameworks
- [Linux Hardening Guide](https://wiki.archlinux.org/title/Security) - Arch Wiki
- [OpenSCAP](https://www.open-scap.org/) - Security compliance toolkit

---

*Last Updated: April 2026*

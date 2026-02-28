# Chapter 5: Security Hardening

[← Previous: Backup & Recovery](04-backup-and-recovery.md) | [Back to README](../README.md) | [Next: SELinux & AppArmor →](06-selinux-apparmor.md)

---

## Overview

Security hardening is the process of reducing the attack surface of a Linux system by disabling unnecessary services, applying patches, enforcing access controls, and monitoring for threats. This chapter covers essential hardening techniques used in DevOps environments to secure servers, containers, and CI/CD pipelines.

---

## 5.1 Security Hardening Mindset

```
┌──────────────────────────────────────────────────────────────┐
│              DEFENSE IN DEPTH                                 │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌─────────────────────────────────────┐                     │
│  │ Layer 1: NETWORK                    │                     │
│  │  Firewalls, VPN, Network Segments   │                     │
│  │  ┌─────────────────────────────┐    │                     │
│  │  │ Layer 2: HOST               │    │                     │
│  │  │  SSH hardening, Patches     │    │                     │
│  │  │  ┌─────────────────────┐    │    │                     │
│  │  │  │ Layer 3: ACCESS     │    │    │                     │
│  │  │  │  Users, sudo, perms │    │    │                     │
│  │  │  │  ┌─────────────┐    │    │    │                     │
│  │  │  │  │ Layer 4:    │    │    │    │                     │
│  │  │  │  │ DATA        │    │    │    │                     │
│  │  │  │  │ Encryption  │    │    │    │                     │
│  │  │  │  │ Backups     │    │    │    │                     │
│  │  │  │  └─────────────┘    │    │    │                     │
│  │  │  └─────────────────────┘    │    │                     │
│  │  └─────────────────────────────┘    │                     │
│  └─────────────────────────────────────┘                     │
│                                                               │
│  Layer 5: MONITORING & AUDITING (wraps everything)           │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 SSH Hardening

### `/etc/ssh/sshd_config`

```bash
# Change default port
Port 2222

# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no
PubkeyAuthentication yes

# Limit users
AllowUsers deploy admin
# Or by group:
AllowGroups sshusers

# Timeout idle sessions
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable unused features
X11Forwarding no
PermitEmptyPasswords no
MaxAuthTries 3

# Use only SSH protocol 2
Protocol 2

# Restrict key exchange algorithms (modern only)
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

```bash
# Apply changes
sudo sshd -t                  # Test config first!
sudo systemctl restart sshd

# Generate strong SSH key
ssh-keygen -t ed25519 -a 100 -C "user@server"
```

---

## 5.3 Firewall Configuration

```bash
# UFW (Ubuntu/Debian)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2222/tcp comment 'SSH'
sudo ufw allow 80/tcp comment 'HTTP'
sudo ufw allow 443/tcp comment 'HTTPS'
sudo ufw enable
sudo ufw status verbose

# firewalld (RHEL/CentOS)
sudo firewall-cmd --set-default-zone=drop
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

---

## 5.4 Fail2Ban — Intrusion Prevention

```bash
# Install
sudo apt install fail2ban       # Ubuntu
sudo dnf install fail2ban       # RHEL (EPEL)

# Create local config (never edit jail.conf directly)
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### `/etc/fail2ban/jail.local`

```ini
[DEFAULT]
bantime  = 3600        # 1 hour ban
findtime = 600         # 10 minute window
maxretry = 3           # 3 failures = ban
banaction = ufw        # or iptables

[sshd]
enabled  = true
port     = 2222
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
bantime  = 86400       # 24 hours for SSH

[nginx-http-auth]
enabled  = true
filter   = nginx-http-auth
logpath  = /var/log/nginx/error.log

[nginx-limit-req]
enabled  = true
filter   = nginx-limit-req
logpath  = /var/log/nginx/error.log
```

```bash
# Manage fail2ban
sudo systemctl enable --now fail2ban
sudo fail2ban-client status
sudo fail2ban-client status sshd

# Unban an IP
sudo fail2ban-client set sshd unbanip 192.168.1.100

# Check banned IPs
sudo fail2ban-client get sshd banned
```

---

## 5.5 Password & Authentication Policies

```bash
# Password complexity (PAM)
# /etc/pam.d/common-password (Ubuntu)
password requisite pam_pwquality.so retry=3 \
    minlen=12 dcredit=-1 ucredit=-1 lcredit=-1 ocredit=-1

# Password aging
sudo chage -M 90 -m 7 -W 14 username
#   -M 90   Max days between changes
#   -m 7    Min days between changes
#   -W 14   Warning days before expiry

# View password policy
sudo chage -l username

# Lock/Unlock accounts
sudo passwd -l username            # Lock
sudo passwd -u username            # Unlock

# Set default aging in /etc/login.defs
PASS_MAX_DAYS   90
PASS_MIN_DAYS   7
PASS_MIN_LEN    12
PASS_WARN_AGE   14
```

---

## 5.6 System Patching

```bash
# Ubuntu/Debian
sudo apt update && sudo apt upgrade -y
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades

# RHEL/CentOS
sudo dnf check-update
sudo dnf upgrade -y
sudo dnf install dnf-automatic
sudo systemctl enable --now dnf-automatic.timer

# Check for security updates only (Ubuntu)
sudo apt list --upgradable 2>/dev/null | grep -i security

# Kernel live patching (Ubuntu)
sudo canonical-livepatch enable TOKEN
```

### Unattended Upgrades Config

```bash
# /etc/apt/apt.conf.d/50unattended-upgrades
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "03:00";
Unattended-Upgrade::Mail "admin@example.com";
```

---

## 5.7 Disable Unnecessary Services

```bash
# List all running services
systemctl list-units --type=service --state=running

# Check listening ports
ss -tlnp

# Disable unnecessary services
sudo systemctl disable --now avahi-daemon    # mDNS
sudo systemctl disable --now cups            # Printing
sudo systemctl disable --now bluetooth       # Bluetooth
sudo systemctl disable --now rpcbind         # NFS RPC

# Mask services (prevent starting even manually)
sudo systemctl mask telnet.socket
```

---

## 5.8 File System Security

```bash
# Set sticky bit on /tmp
chmod 1777 /tmp

# Secure mount options in /etc/fstab
/dev/sdb1  /tmp      ext4  defaults,nosuid,noexec,nodev  0 2
/dev/sdb2  /var/log  ext4  defaults,nosuid,noexec,nodev  0 2

# Find world-writable files
find / -type f -perm -002 -ls 2>/dev/null

# Find SUID binaries (potential privilege escalation)
find / -type f -perm -4000 -ls 2>/dev/null

# Find SGID binaries
find / -type f -perm -2000 -ls 2>/dev/null

# Find files with no owner
find / -nouser -o -nogroup 2>/dev/null

# Set immutable attribute (even root can't modify)
sudo chattr +i /etc/passwd
sudo lsattr /etc/passwd
```

---

## 5.9 Audit Framework (auditd)

```bash
# Install
sudo apt install auditd          # Ubuntu
sudo dnf install audit            # RHEL

# Enable
sudo systemctl enable --now auditd
```

### Audit Rules

```bash
# /etc/audit/rules.d/hardening.rules

# Monitor password file changes
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudoers

# Monitor SSH config changes
-w /etc/ssh/sshd_config -p wa -k sshd_config

# Monitor cron
-w /etc/crontab -p wa -k cron
-w /var/spool/cron/ -p wa -k cron

# Log all commands run as root
-a always,exit -F arch=b64 -F euid=0 -S execve -k root_commands

# Monitor Docker socket
-w /var/run/docker.sock -p wa -k docker
```

```bash
# Reload rules
sudo augenrules --load

# Search audit logs
sudo ausearch -k identity              # By key
sudo ausearch -m USER_LOGIN            # By message type
sudo ausearch -ui 1000                 # By user ID

# Generate reports
sudo aureport --login                  # Login report
sudo aureport --summary                # Summary
sudo aureport --failed                 # Failed events
```

---

## 5.10 Network Security

```bash
# Disable IP source routing
echo "net.ipv4.conf.all.accept_source_route = 0" >> /etc/sysctl.d/99-security.conf
echo "net.ipv4.conf.default.accept_source_route = 0" >> /etc/sysctl.d/99-security.conf

# Disable ICMP redirects
echo "net.ipv4.conf.all.accept_redirects = 0" >> /etc/sysctl.d/99-security.conf

# Enable SYN cookies (SYN flood protection)
echo "net.ipv4.tcp_syncookies = 1" >> /etc/sysctl.d/99-security.conf

# Ignore ping (optional, cautious in DevOps)
echo "net.ipv4.icmp_echo_ignore_all = 1" >> /etc/sysctl.d/99-security.conf

# Log suspicious packets
echo "net.ipv4.conf.all.log_martians = 1" >> /etc/sysctl.d/99-security.conf

# Apply
sudo sysctl --system
```

---

## 5.11 CIS Benchmark Quick Checklist

```
┌──────────────────────────────────────────────────────────────┐
│         CIS HARDENING CHECKLIST (TOP ITEMS)                   │
├───┬──────────────────────────────────────────────────────────┤
│ # │ Action                                                   │
├───┼──────────────────────────────────────────────────────────┤
│ 1 │ Ensure updates are installed                             │
│ 2 │ Disable unused filesystems (cramfs, squashfs, udf)       │
│ 3 │ Set noexec,nosuid,nodev on /tmp                         │
│ 4 │ Ensure AIDE is installed (file integrity monitoring)     │
│ 5 │ Set boot loader password (GRUB)                         │
│ 6 │ Restrict core dumps                                      │
│ 7 │ Enable ASLR                                              │
│ 8 │ Disable root login over SSH                              │
│ 9 │ Set password policies                                    │
│10 │ Configure audit logging                                  │
│11 │ Restrict su access                                       │
│12 │ Disable IPv6 if not used                                 │
│13 │ Set umask to 027                                         │
│14 │ Remove legacy services (telnet, rsh)                     │
│15 │ Configure NTP time sync                                  │
└───┴──────────────────────────────────────────────────────────┘
```

---

## 5.12 Real-World Scenario: Server Hardening Script

```bash
#!/usr/bin/env bash
# harden.sh — Automated security hardening for new servers
# Usage: sudo ./harden.sh

set -euo pipefail

LOG="/var/log/hardening.log"
log() { echo "$(date '+%F %T') [HARDEN] $*" | tee -a "$LOG"; }

if [ "$(id -u)" -ne 0 ]; then
    echo "Must run as root" >&2
    exit 1
fi

log "=== Starting Security Hardening ==="

# 1. Update system
log "Updating packages..."
if command -v apt &>/dev/null; then
    apt update -qq && apt upgrade -y -qq
elif command -v dnf &>/dev/null; then
    dnf upgrade -y -q
fi
log "System updated"

# 2. SSH Hardening
log "Hardening SSH..."
SSHD_CONFIG="/etc/ssh/sshd_config"
cp "$SSHD_CONFIG" "${SSHD_CONFIG}.bak"

sed -i 's/^#\?PermitRootLogin.*/PermitRootLogin no/' "$SSHD_CONFIG"
sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' "$SSHD_CONFIG"
sed -i 's/^#\?X11Forwarding.*/X11Forwarding no/' "$SSHD_CONFIG"
sed -i 's/^#\?MaxAuthTries.*/MaxAuthTries 3/' "$SSHD_CONFIG"
sed -i 's/^#\?PermitEmptyPasswords.*/PermitEmptyPasswords no/' "$SSHD_CONFIG"

# Add idle timeout if not set
grep -q "ClientAliveInterval" "$SSHD_CONFIG" || \
    echo -e "\nClientAliveInterval 300\nClientAliveCountMax 2" >> "$SSHD_CONFIG"

sshd -t && systemctl restart sshd
log "SSH hardened (root login disabled, password auth disabled)"

# 3. Firewall
log "Configuring firewall..."
if command -v ufw &>/dev/null; then
    ufw default deny incoming
    ufw default allow outgoing
    ufw allow 22/tcp comment 'SSH'
    ufw allow 80/tcp comment 'HTTP'
    ufw allow 443/tcp comment 'HTTPS'
    echo "y" | ufw enable
    log "UFW configured"
elif command -v firewall-cmd &>/dev/null; then
    firewall-cmd --set-default-zone=drop
    firewall-cmd --permanent --add-service=ssh
    firewall-cmd --permanent --add-service=http
    firewall-cmd --permanent --add-service=https
    firewall-cmd --reload
    log "firewalld configured"
fi

# 4. Install fail2ban
log "Installing fail2ban..."
if command -v apt &>/dev/null; then
    apt install -y -qq fail2ban
else
    dnf install -y -q fail2ban
fi
cat > /etc/fail2ban/jail.local <<'EOF'
[DEFAULT]
bantime  = 3600
findtime = 600
maxretry = 3

[sshd]
enabled  = true
maxretry = 3
bantime  = 86400
EOF
systemctl enable --now fail2ban
log "Fail2ban installed and configured"

# 5. Kernel hardening
log "Applying kernel security parameters..."
cat > /etc/sysctl.d/99-security.conf <<'EOF'
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv6.conf.all.accept_redirects = 0
kernel.randomize_va_space = 2
fs.suid_dumpable = 0
EOF
sysctl --system > /dev/null 2>&1
log "Kernel parameters hardened"

# 6. Disable unnecessary services
log "Disabling unnecessary services..."
for svc in avahi-daemon cups bluetooth rpcbind; do
    if systemctl is-active "$svc" &>/dev/null; then
        systemctl disable --now "$svc"
        log "Disabled: $svc"
    fi
done

# 7. Set password policies
log "Setting password policies..."
if [ -f /etc/login.defs ]; then
    sed -i 's/^PASS_MAX_DAYS.*/PASS_MAX_DAYS   90/' /etc/login.defs
    sed -i 's/^PASS_MIN_DAYS.*/PASS_MIN_DAYS   7/' /etc/login.defs
    sed -i 's/^PASS_WARN_AGE.*/PASS_WARN_AGE   14/' /etc/login.defs
fi
log "Password policies configured"

# 8. Restrict core dumps
log "Restricting core dumps..."
echo "* hard core 0" >> /etc/security/limits.conf
echo "fs.suid_dumpable = 0" >> /etc/sysctl.d/99-security.conf
sysctl -p /etc/sysctl.d/99-security.conf > /dev/null 2>&1
log "Core dumps restricted"

# 9. Setup audit rules
log "Configuring audit rules..."
if command -v auditctl &>/dev/null; then
    cat > /etc/audit/rules.d/hardening.rules <<'EOF'
-w /etc/passwd -p wa -k identity
-w /etc/shadow -p wa -k identity
-w /etc/group -p wa -k identity
-w /etc/sudoers -p wa -k sudoers
-w /etc/ssh/sshd_config -p wa -k sshd_config
EOF
    augenrules --load 2>/dev/null
    log "Audit rules configured"
fi

# 10. Set secure umask
log "Setting secure umask..."
echo "umask 027" >> /etc/profile

# Summary
log "=== Hardening Complete ==="
log "Actions taken:"
log "  - System updated"
log "  - SSH hardened (no root, no password, timeout)"
log "  - Firewall configured (deny incoming by default)"
log "  - Fail2ban installed"
log "  - Kernel parameters hardened"
log "  - Unnecessary services disabled"
log "  - Password policies set"
log "  - Core dumps restricted"
log "  - Audit rules configured"
log "  - Secure umask set"
log ""
log "IMPORTANT: Ensure you have SSH key access before disconnecting!"
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| Locked out via SSH | Too strict config | Use console/KVM access, check `sshd_config` |
| Fail2ban blocked legitimate IP | Too low `maxretry` | `fail2ban-client set sshd unbanip IP` |
| Service won't start after hardening | Disabled dependency | Check `journalctl -xe`, re-enable needed service |
| SELinux denying app | Policy violation | Check `audit2why`, create custom policy (next chapter) |
| Users can't change passwords | PAM misconfigured | Test `pam_pwquality` settings, check `/var/log/auth.log` |
| Unattended upgrades not running | Timer inactive | `systemctl enable --now unattended-upgrades` |

---

## Summary Table

| Area | Tool/Config | Key Action |
|------|-------------|------------|
| SSH | `sshd_config` | Disable root, keys only, limit users |
| Firewall | `ufw`/`firewalld` | Default deny, allow only needed ports |
| Intrusion prevention | `fail2ban` | Auto-ban brute force attempts |
| Patching | `unattended-upgrades` | Automatic security updates |
| Passwords | PAM, `chage` | Min length, complexity, aging |
| Attack surface | `systemctl` | Disable unused services |
| File integrity | `AIDE`, `auditd` | Monitor critical file changes |
| Network | `sysctl` | Disable source routing, SYN cookies |
| Access | `sudo`, permissions | Principle of least privilege |

---

## Quick Revision Questions

1. **What SSH configuration changes should you make on a production server?**
2. **How does Fail2Ban protect against brute force attacks?**
3. **What is the purpose of the `auditd` daemon and audit rules?**
4. **Why should you use `sshd -t` before restarting the SSH service?**
5. **What kernel parameters help protect against network attacks?**
6. **What is a CIS benchmark and why is it important for DevOps?**

---

[← Previous: Backup & Recovery](04-backup-and-recovery.md) | [Back to README](../README.md) | [Next: SELinux & AppArmor →](06-selinux-apparmor.md)

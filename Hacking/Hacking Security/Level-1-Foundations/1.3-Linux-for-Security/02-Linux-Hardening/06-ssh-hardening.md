# SSH Hardening

## Unit 2 - Topic 6 | Linux Hardening

---

## Overview

**SSH (Secure Shell)** is the primary remote access method for Linux servers. Because it provides **direct shell access**, SSH is a high-value target for attackers. Hardening SSH is one of the most critical steps in Linux security — a compromised SSH service means complete system access.

---

## 1. SSH Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              SSH CONNECTION FLOW                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client                          Server                      │
│  ┌─────────┐                     ┌─────────┐                │
│  │ ssh     │ ──── TCP 22 ──────► │ sshd    │                │
│  │ client  │                     │ daemon  │                │
│  └────┬────┘                     └────┬────┘                │
│       │                               │                      │
│  1. Key Exchange (Diffie-Hellman)                            │
│  2. Server Authentication (host key)                         │
│  3. User Authentication:                                     │
│     a. Public key (preferred)                                │
│     b. Password (less secure)                                │
│     c. Keyboard-interactive (MFA)                            │
│  4. Encrypted session established                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. SSH Hardening Configuration

```bash
# /etc/ssh/sshd_config — CRITICAL SETTINGS

# === AUTHENTICATION ===
# Disable root login
PermitRootLogin no

# Use SSH protocol 2 only (protocol 1 is broken)
Protocol 2

# Disable password authentication (use keys only)
PasswordAuthentication no
ChallengeResponseAuthentication no

# Enable public key authentication
PubkeyAuthentication yes

# Disable empty passwords
PermitEmptyPasswords no

# === ACCESS CONTROL ===
# Allow only specific users
AllowUsers alice bob
# Or allow specific groups
AllowGroups sshusers admins

# Deny specific users
DenyUsers root guest

# === NETWORK ===
# Change default port (security through obscurity — helps with bots)
Port 2222

# Listen on specific interface only
ListenAddress 10.0.0.5

# Limit connections
MaxAuthTries 3
MaxSessions 5
LoginGraceTime 30

# === SESSION ===
# Disable X11 forwarding (unless needed)
X11Forwarding no

# Disable TCP forwarding (unless needed)
AllowTcpForwarding no

# Disable agent forwarding
AllowAgentForwarding no

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# === CRYPTO ===
# Use strong ciphers only
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com,aes256-ctr
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
KexAlgorithms curve25519-sha256,diffie-hellman-group16-sha512

# === LOGGING ===
LogLevel VERBOSE

# === MISCELLANEOUS ===
# Disable motd banner info leak
PrintMotd no
# Show custom banner
Banner /etc/ssh/banner.txt
# Disable .rhosts files
IgnoreRhosts yes
# Disable host-based authentication
HostbasedAuthentication no
```

---

## 3. SSH Key Authentication Setup

```bash
# ON CLIENT — Generate key pair
ssh-keygen -t ed25519 -C "alice@workstation"
# Or RSA (4096-bit minimum)
ssh-keygen -t rsa -b 4096 -C "alice@workstation"

# Copy public key to server
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server

# OR manually:
cat ~/.ssh/id_ed25519.pub | ssh alice@server \
  'mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys'

# ON SERVER — Set permissions
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chown -R alice:alice ~/.ssh

# After verifying key login works, disable passwords:
# PasswordAuthentication no
# Then restart: systemctl restart sshd
```

---

## 4. SSH Attack Vectors

| Attack | Description | Defense |
|--------|-------------|---------|
| **Brute force** | Automated password guessing | Key auth, fail2ban, MaxAuthTries |
| **Credential stuffing** | Stolen credentials from breaches | Key auth, MFA |
| **MITM** | Intercept SSH connection | Verify host key fingerprints |
| **Key theft** | Steal private key file | Passphrase on keys, hardware keys |
| **Default port scan** | Automated scanning of port 22 | Change port, rate limit |

---

## 5. fail2ban — Brute Force Protection

```bash
# Install
apt install fail2ban

# /etc/fail2ban/jail.local
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600            # Ban for 1 hour
findtime = 600            # Within 10 minutes

# Start and enable
systemctl enable --now fail2ban

# Check status
fail2ban-client status sshd

# Unban an IP
fail2ban-client set sshd unbanip 10.0.0.100
```

---

## 6. SSH Hardening Checklist

| # | Check | Setting |
|:-:|-------|---------|
| 1 | Disable root login | `PermitRootLogin no` |
| 2 | Key-only authentication | `PasswordAuthentication no` |
| 3 | Strong ciphers | AES-256-GCM, ChaCha20 |
| 4 | Limit users | `AllowUsers` / `AllowGroups` |
| 5 | Reduce auth attempts | `MaxAuthTries 3` |
| 6 | Set idle timeout | `ClientAliveInterval 300` |
| 7 | Disable forwarding | `X11Forwarding no` |
| 8 | Install fail2ban | Rate-limit login attempts |
| 9 | Use Ed25519 keys | Stronger than RSA-2048 |
| 10 | Verbose logging | `LogLevel VERBOSE` |

```bash
# After changes, test config before restarting
sshd -t                              # Test configuration
systemctl restart sshd               # Apply changes
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Root login** | Always disable (`PermitRootLogin no`) |
| **Key auth** | Use Ed25519 keys, disable passwords |
| **fail2ban** | Auto-ban IPs after failed attempts |
| **Strong crypto** | AES-256-GCM, ChaCha20-Poly1305 |
| **Access control** | AllowUsers/AllowGroups whitelist |
| **Test first** | `sshd -t` before restarting |

---

## Quick Revision Questions

1. **Why should root login be disabled via SSH?**
2. **What is the most secure SSH key type?**
3. **How does fail2ban protect SSH?**
4. **What config change disables password authentication?**
5. **How do you test SSH config before restarting the service?**

---

[← Previous: PAM Configuration](05-pam-configuration.md) | [Next: Firewall Configuration →](07-firewall-configuration.md)

---

*Unit 2 - Topic 6 of 7 | [Back to README](../README.md)*

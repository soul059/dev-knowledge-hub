# SSH (Secure Shell)

## Unit 5 - Topic 5 | Application Layer Protocols

---

## Overview

**SSH (Secure Shell)** provides encrypted remote access to systems, replacing insecure protocols like Telnet and rlogin. SSH is the backbone of secure system administration, supporting **remote command execution**, **file transfer (SFTP/SCP)**, **port forwarding**, and **tunneling**. It's also a prime target for brute force attacks.

---

## 1. How SSH Works

```
┌──────────────────────────────────────────────────────────────────┐
│                  SSH CONNECTION FLOW                             │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Client                                     Server (port 22)    │
│    │                                           │                │
│    │ 1. TCP Connection (port 22)               │                │
│    │──────────────────────────────────────────►│                │
│    │                                           │                │
│    │ 2. Protocol Version Exchange              │                │
│    │◄────────────────────────────────────────►│                │
│    │                                           │                │
│    │ 3. Key Exchange (Diffie-Hellman)           │                │
│    │◄════════════════════════════════════════►│                │
│    │  → Shared session key established          │                │
│    │  → All traffic now ENCRYPTED               │                │
│    │                                           │                │
│    │ 4. Server Authentication                   │                │
│    │  Client verifies server's host key         │                │
│    │                                           │                │
│    │ 5. User Authentication                     │                │
│    │  Password OR SSH key OR certificate        │                │
│    │──────────────────────────────────────────►│                │
│    │                                           │                │
│    │ 6. Encrypted Session Established           │                │
│    │◄════════════════════════════════════════►│                │
│    │                                           │                │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. SSH Authentication Methods

| Method | Security | Description |
|--------|:--------:|-------------|
| **Password** | ⚠️ Medium | Typed password — vulnerable to brute force |
| **Public Key** | ✅ Strong | Key pair (private on client, public on server) |
| **Certificate** | ✅ Strongest | CA-signed SSH certificates |
| **Keyboard-Interactive** | ⚠️ Medium | Multi-step auth (can include MFA) |

### SSH Key Setup

```bash
# Generate SSH key pair (Ed25519 — recommended)
ssh-keygen -t ed25519 -C "user@host"

# Copy public key to server
ssh-copy-id user@server

# Connect using key
ssh -i ~/.ssh/id_ed25519 user@server

# Key files:
# ~/.ssh/id_ed25519       ← Private key (NEVER share!)
# ~/.ssh/id_ed25519.pub   ← Public key (copy to server)
# ~/.ssh/authorized_keys  ← Server's list of allowed public keys
# ~/.ssh/known_hosts      ← Client's record of server fingerprints
```

---

## 3. SSH Attacks

| Attack | Description | Defense |
|--------|-------------|---------|
| **Brute Force** | Try many passwords | Key-based auth, fail2ban |
| **Key Theft** | Steal private key from client | Passphrase on key, secure storage |
| **MITM** | Intercept SSH connection (first connect) | Verify host key fingerprint |
| **SSH Downgrade** | Force weak cipher/protocol | Configure strong ciphers only |
| **Credential Stuffing** | Use leaked passwords | Unique passwords, key auth |

```bash
# SSH brute force with Hydra
hydra -l root -P wordlist.txt ssh://target.com

# SSH brute force with Nmap
nmap -p 22 --script ssh-brute target.com

# Defense: fail2ban configuration
# /etc/fail2ban/jail.local
[sshd]
enabled = true
maxretry = 3
bantime = 3600
```

---

## 4. SSH Hardening (sshd_config)

```bash
# /etc/ssh/sshd_config — Key hardening settings

# Disable root login
PermitRootLogin no

# Disable password authentication (key-only)
PasswordAuthentication no
PubkeyAuthentication yes

# Change default port (security through obscurity + reduces noise)
Port 2222

# Limit users
AllowUsers admin deploy
AllowGroups sshusers

# Disable empty passwords
PermitEmptyPasswords no

# Set idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Use SSH Protocol 2 only
Protocol 2

# Strong ciphers only
Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
MACs hmac-sha2-256-etm@openssh.com,hmac-sha2-512-etm@openssh.com
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org

# Restart SSH
sudo systemctl restart sshd
```

---

## 5. SSH Tunneling & Port Forwarding

```bash
# LOCAL Port Forwarding (access remote service through local port)
ssh -L 8080:internal-server:80 user@jumphost
# Now http://localhost:8080 → reaches internal-server:80

# REMOTE Port Forwarding (expose local service to remote)
ssh -R 9090:localhost:3000 user@remote-server
# Remote server's port 9090 → reaches your localhost:3000

# DYNAMIC Port Forwarding (SOCKS proxy)
ssh -D 1080 user@jumphost
# Configure browser to use SOCKS5 proxy localhost:1080
# All browser traffic tunneled through jumphost

# SSH as VPN — browse internal network through SSH tunnel
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SSH** | Encrypted remote access — replaces Telnet |
| **Best Auth** | Ed25519 key-based authentication |
| **Top Threat** | Brute force attacks on password auth |
| **Key Hardening** | Disable root login, disable passwords, strong ciphers |
| **Tunneling** | Local, Remote, Dynamic port forwarding |
| **Port** | 22 (default) — consider changing |

---

## Quick Revision Questions

1. **Why is SSH more secure than Telnet?**
2. **What is the most secure SSH authentication method?**
3. **What SSH hardening settings should be applied?**
4. **How does SSH local port forwarding work?**
5. **What is fail2ban and how does it protect SSH?**
6. **What is the difference between Ed25519 and RSA SSH keys?**

---

[← Previous: FTP/SFTP/SCP](04-ftp-sftp-scp.md) | [Next: SNMP Security →](06-snmp-security.md)

---

*Unit 5 - Topic 5 of 6 | [Back to README](../README.md)*

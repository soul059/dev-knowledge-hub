# Chapter 5: SSH

## Overview

SSH (Secure Shell) is the primary protocol for securely accessing remote servers. It provides encrypted communication, remote command execution, file transfer, and tunneling. SSH is arguably the most-used tool in a DevOps engineer's daily workflow — from managing servers to running Ansible playbooks to accessing Git repositories.

---

## 5.1 How SSH Works

```
┌──────────────────────────────────────────────────────────────┐
│                  SSH CONNECTION FLOW                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client                                    Server            │
│  (your laptop)                             (remote host)     │
│    │                                          │              │
│    │── TCP Connection (port 22) ────────────▶│              │
│    │                                          │              │
│    │◀─ Server sends public key ──────────────│              │
│    │   (host key fingerprint)                 │              │
│    │                                          │              │
│    │── Key exchange (Diffie-Hellman) ───────▶│              │
│    │   (establish shared secret)              │              │
│    │                                          │              │
│    │── Authentication ──────────────────────▶│              │
│    │   Option A: Password                     │              │
│    │   Option B: Public key (preferred)       │              │
│    │   Option C: Certificate-based            │              │
│    │                                          │              │
│    │◀═══ ENCRYPTED SESSION ═════════════════▶│              │
│    │     (all traffic encrypted)              │              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.2 SSH Key-Based Authentication

```
┌──────────────────────────────────────────────────────────────┐
│           SSH KEY-BASED AUTH (How it Works)                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SETUP (one-time):                                           │
│  ┌──────────┐                    ┌──────────────────┐        │
│  │  Client  │   Copy public key  │     Server       │        │
│  │  ────────│   ──────────────▶  │  ~/.ssh/          │        │
│  │  Private │                    │  authorized_keys  │        │
│  │  Key 🔑  │                    │  (public key) 🔓  │        │
│  └──────────┘                    └──────────────────┘        │
│                                                              │
│  LOGIN (each time):                                          │
│  ┌──────────┐                    ┌──────────────────┐        │
│  │  Client  │── "I have the  ──▶│     Server       │        │
│  │          │   private key"     │                  │        │
│  │          │                    │  Verifies against│        │
│  │          │◀── Challenge ──────│  authorized_keys │        │
│  │          │                    │                  │        │
│  │          │── Signed ────────▶│  ✓ Match!        │        │
│  │          │   Response         │  Access Granted  │        │
│  └──────────┘                    └──────────────────┘        │
│                                                              │
│  Private key NEVER leaves the client!                        │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Generate SSH Keys

```bash
# Generate RSA key pair (4096-bit)
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

# Generate Ed25519 key (recommended — faster, more secure)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Generated files:
# ~/.ssh/id_ed25519       ← Private key (KEEP SECRET!)
# ~/.ssh/id_ed25519.pub   ← Public key (share freely)

# Copy public key to remote server
ssh-copy-id user@remote-server

# Or manually:
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Set correct permissions (critical!)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

## 5.3 SSH Configuration File

The SSH config file (`~/.ssh/config`) simplifies connections:

```bash
# ~/.ssh/config

# Production web server
Host prod-web
    HostName 54.210.33.100
    User ubuntu
    IdentityFile ~/.ssh/prod-key.pem
    Port 22

# Staging server via jump/bastion host
Host staging
    HostName 10.0.2.50
    User ec2-user
    IdentityFile ~/.ssh/staging-key.pem
    ProxyJump bastion

# Bastion/Jump host
Host bastion
    HostName 54.100.200.50
    User ec2-user
    IdentityFile ~/.ssh/bastion-key.pem

# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    StrictHostKeyChecking ask
    AddKeysToAgent yes

# Usage: just type
# ssh prod-web        (instead of ssh -i ~/.ssh/prod-key.pem ubuntu@54.210.33.100)
# ssh staging         (automatically goes through bastion)
```

---

## 5.4 SSH Tunneling (Port Forwarding)

### Local Port Forwarding

Access a remote service through a local port:

```
┌──────────────────────────────────────────────────────────────┐
│           LOCAL PORT FORWARDING (-L)                         │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  You want to access DB on 10.0.2.50:5432 from your laptop   │
│  but it's in a private subnet.                               │
│                                                              │
│  ┌────────┐         ┌──────────┐         ┌──────────┐       │
│  │ Laptop │──SSH───▶│ Bastion  │────────▶│ Database │       │
│  │:9999   │ tunnel  │ (public) │         │ 10.0.2.50│       │
│  └────────┘         └──────────┘         │ :5432    │       │
│                                          └──────────┘       │
│  localhost:9999 ══tunnel══▶ 10.0.2.50:5432                   │
│                                                              │
└──────────────────────────────────────────────────────────────┘

# Command:
ssh -L 9999:10.0.2.50:5432 user@bastion

# Now connect to DB via:
psql -h localhost -p 9999 -U dbuser mydb
```

### Remote Port Forwarding

Expose a local service to the remote network:

```
┌──────────────────────────────────────────────────────────────┐
│           REMOTE PORT FORWARDING (-R)                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Your local dev server (localhost:3000) needs to be          │
│  accessible from the remote network.                         │
│                                                              │
│  ┌────────┐         ┌──────────┐                            │
│  │ Laptop │──SSH───▶│  Remote  │                            │
│  │ :3000  │ tunnel  │  Server  │                            │
│  └────────┘         │  :8080   │                            │
│                     └──────────┘                            │
│  remote:8080 ══tunnel══▶ localhost:3000                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘

# Command:
ssh -R 8080:localhost:3000 user@remote-server

# Now anyone on the remote network can access:
# http://remote-server:8080 → your local app
```

### Dynamic Port Forwarding (SOCKS Proxy)

```bash
# Create a SOCKS proxy through SSH
ssh -D 1080 user@remote-server

# Configure browser/app to use SOCKS proxy: localhost:1080
# All traffic goes through the SSH tunnel
```

---

## 5.5 SSH in DevOps Workflows

### Ansible

```yaml
# ansible.cfg
[defaults]
remote_user = ubuntu
private_key_file = ~/.ssh/prod-key.pem
host_key_checking = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
```

### Git over SSH

```bash
# Test SSH connection to GitHub
ssh -T git@github.com

# Clone via SSH
git clone git@github.com:user/repo.git

# Add SSH key to ssh-agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# ~/.ssh/config for GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_ed25519
```

### SCP and SFTP

```bash
# Copy file to remote server (SCP)
scp file.txt user@server:/tmp/

# Copy directory recursively
scp -r ./project user@server:/opt/

# Interactive SFTP session
sftp user@server
sftp> put localfile.txt
sftp> get remotefile.txt
sftp> ls
sftp> exit

# Using rsync over SSH (preferred for large transfers)
rsync -avz -e ssh ./project/ user@server:/opt/project/
```

---

## 5.6 SSH Security Best Practices

```
┌──────────────────────────────────────────────────────────────┐
│              SSH HARDENING CHECKLIST                          │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  /etc/ssh/sshd_config:                                       │
│                                                              │
│  ✓ Disable root login                                        │
│    PermitRootLogin no                                        │
│                                                              │
│  ✓ Disable password authentication                           │
│    PasswordAuthentication no                                 │
│                                                              │
│  ✓ Use key-based auth only                                   │
│    PubkeyAuthentication yes                                  │
│                                                              │
│  ✓ Change default port (optional, security by obscurity)     │
│    Port 2222                                                 │
│                                                              │
│  ✓ Limit users                                               │
│    AllowUsers deploy admin                                   │
│                                                              │
│  ✓ Set idle timeout                                          │
│    ClientAliveInterval 300                                   │
│    ClientAliveCountMax 2                                     │
│                                                              │
│  ✓ Disable empty passwords                                   │
│    PermitEmptyPasswords no                                   │
│                                                              │
│  ✓ Use strong ciphers only                                   │
│    Ciphers aes256-gcm@openssh.com,aes128-gcm@openssh.com    │
│                                                              │
│  After changes:                                              │
│  $ sudo systemctl restart sshd                               │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Bastion Host Pattern

```
┌──────────────────────────────────────────────────────────────┐
│              BASTION HOST ARCHITECTURE                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Internet                                                    │
│     │                                                        │
│     ▼                                                        │
│  ┌──────────────────── VPC ──────────────────────────┐       │
│  │                                                    │       │
│  │  Public Subnet                                     │       │
│  │  ┌─────────────────┐                               │       │
│  │  │  Bastion Host   │  ← Only SSH exposed           │       │
│  │  │  (Jump Box)     │     to internet               │       │
│  │  │  SG: port 22    │                               │       │
│  │  │  from your IP   │                               │       │
│  │  └────────┬────────┘                               │       │
│  │           │ SSH                                    │       │
│  │  Private Subnet                                    │       │
│  │  ┌────────▼────────┐  ┌─────────────┐             │       │
│  │  │  App Server     │  │ DB Server   │             │       │
│  │  │  SG: port 22    │  │ SG: port 22 │             │       │
│  │  │  from bastion   │  │ from bastion│             │       │
│  │  │  only           │  │ only        │             │       │
│  │  └─────────────────┘  └─────────────┘             │       │
│  │                                                    │       │
│  └────────────────────────────────────────────────────┘       │
│                                                              │
│  Modern alternative: AWS Systems Manager Session Manager     │
│  (no bastion needed, no SSH port exposed)                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 5.7 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| "Connection refused" | SSHD not running or wrong port | `systemctl status sshd`, check port |
| "Permission denied (publickey)" | Wrong key or permissions | Check key path, `chmod 600` on key |
| "Host key verification failed" | Server key changed | Remove old key from `~/.ssh/known_hosts` |
| Connection timeout | Firewall/SG blocking port 22 | Check security group, NACL, firewall |
| "Too many authentication failures" | SSH agent trying all keys | Use `ssh -i specific_key` or config |
| Slow SSH login | Reverse DNS lookup | Add `UseDNS no` to sshd_config |

```bash
# Debug SSH connection issues
ssh -vvv user@server

# Check if SSH port is open
nc -zv server 22
telnet server 22

# Check SSH service
systemctl status sshd
journalctl -u sshd -f
```

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| SSH | Encrypted remote access on port 22 (TCP) |
| Key Auth | Private key (client) + public key (server's authorized_keys) |
| Ed25519 | Recommended key type (faster, more secure than RSA) |
| SSH Config | `~/.ssh/config` simplifies connection management |
| Local Tunnel (-L) | Access remote service via local port |
| Remote Tunnel (-R) | Expose local service to remote network |
| Bastion Host | Single hardened entry point to private network |
| SCP/SFTP | File transfer over SSH |
| Hardening | Disable passwords, root login; use key auth, restrict users |

---

## Quick Revision Questions

1. **Why is key-based SSH authentication more secure than password authentication?**
2. **Write the SSH command to tunnel local port 5432 to a remote database at 10.0.2.50:5432 through a bastion host.**
3. **What SSH config file settings would you use to connect to a server behind a bastion host?**
4. **Name five SSH hardening measures you would apply to a production server.**
5. **What does `ssh -vvv` do, and when would you use it?**
6. **What is the modern alternative to bastion hosts in AWS, and why might you prefer it?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← DHCP](04-dhcp.md) | [README](../README.md) | [FTP/SFTP →](06-ftp-sftp.md) |

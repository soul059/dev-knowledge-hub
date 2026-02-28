# Chapter 6: SSH Configuration

[← Previous: Firewall Configuration](05-firewall-configuration.md) | [Back to README](../README.md) | [Next: Network Troubleshooting →](07-network-troubleshooting.md)

---

## Overview

SSH (Secure Shell) is the primary tool for remote server management in DevOps. This chapter covers SSH key authentication, config files, tunneling, SCP/SFTP, and security hardening — essential skills for managing fleets of servers, CI/CD pipelines, and Git operations.

---

## 6.1 SSH Basics

```
┌──────────────────────────────────────────────────────────────┐
│                SSH CONNECTION FLOW                            │
│                                                                │
│  Client                          Server                       │
│  ┌──────────┐                    ┌──────────────┐            │
│  │ ssh cmd  │──── TCP:22 ───────→│   sshd       │            │
│  └──────────┘                    └──────────────┘            │
│       │                                │                      │
│       │  1. TCP Handshake              │                      │
│       │  2. Server sends host key      │                      │
│       │  3. Key exchange (DH)          │                      │
│       │  4. Encrypted channel est.     │                      │
│       │  5. User authentication        │                      │
│       │     ├── Password               │                      │
│       │     └── Public Key (preferred) │                      │
│       │  6. Shell session              │                      │
│       │←────── Encrypted ─────────────→│                      │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Basic SSH connection
ssh user@hostname
ssh -p 2222 user@hostname            # Custom port
ssh user@192.168.1.100

# Run a single command
ssh user@server "uptime"
ssh user@server "df -h && free -m"

# Verbose (debug connection)
ssh -v user@server                   # Level 1
ssh -vv user@server                  # Level 2
ssh -vvv user@server                 # Level 3 (maximum)
```

---

## 6.2 SSH Key Authentication

```
┌──────────────────────────────────────────────────────────────┐
│             SSH KEY AUTHENTICATION                            │
│                                                                │
│  CLIENT                          SERVER                       │
│  ~/.ssh/id_ed25519  (private)    ~/.ssh/authorized_keys       │
│  ~/.ssh/id_ed25519.pub (public)  (contains public key)        │
│                                                                │
│  Setup:                                                        │
│  1. Generate key pair on CLIENT                               │
│  2. Copy PUBLIC key to SERVER's authorized_keys                │
│  3. SSH uses private key to prove identity                    │
│                                                                │
│  Auth flow:                                                    │
│  Client ──→ "I have key X" ──→ Server                        │
│  Client ←── Challenge ←────── Server                          │
│  Client ──→ Signed response ──→ Server                       │
│  Client ←── Access granted ←── Server (verified with pub key)│
└──────────────────────────────────────────────────────────────┘
```

```bash
# Generate SSH key pair
ssh-keygen -t ed25519 -C "devops@company.com"
# Creates:
#   ~/.ssh/id_ed25519        (private key — NEVER share)
#   ~/.ssh/id_ed25519.pub    (public key — copy to servers)

# Alternative: RSA (wider compatibility)
ssh-keygen -t rsa -b 4096 -C "devops@company.com"

# Copy public key to server
ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/id_ed25519.pub user@server

# Manual copy
cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# Verify permissions (CRITICAL — SSH refuses if wrong)
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519           # Private key
chmod 644 ~/.ssh/id_ed25519.pub       # Public key
chmod 600 ~/.ssh/authorized_keys      # On server
chmod 644 ~/.ssh/known_hosts
```

---

## 6.3 SSH Config File

```bash
# ~/.ssh/config — Client configuration

# Default settings for all hosts
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
    AddKeysToAgent yes

# Production servers
Host prod-web
    HostName 10.0.1.10
    User deploy
    Port 22
    IdentityFile ~/.ssh/prod_key

Host prod-db
    HostName 10.0.1.20
    User deploy
    Port 22
    IdentityFile ~/.ssh/prod_key

# Staging
Host staging
    HostName staging.company.com
    User admin
    IdentityFile ~/.ssh/staging_key

# Jump host (bastion)
Host bastion
    HostName bastion.company.com
    User jump-user
    IdentityFile ~/.ssh/bastion_key

Host internal-*
    ProxyJump bastion
    User admin
    IdentityFile ~/.ssh/internal_key

Host internal-app
    HostName 10.0.2.10

Host internal-db
    HostName 10.0.2.20

# GitHub
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
```

```bash
# Now connect with short names
ssh prod-web                 # Instead of: ssh -i ~/.ssh/prod_key deploy@10.0.1.10
ssh staging
ssh internal-app             # Automatically jumps through bastion
```

---

## 6.4 SSH Tunneling / Port Forwarding

```
┌──────────────────────────────────────────────────────────────┐
│           SSH TUNNEL TYPES                                    │
│                                                                │
│  LOCAL FORWARD (-L):                                          │
│  Access remote service through local port                     │
│  ┌────────┐  :8080  ──SSH──  ┌────────┐  :5432  ┌────────┐  │
│  │ Laptop │──────────────────│ Bastion │─────────│  DB    │  │
│  └────────┘                  └────────┘          └────────┘  │
│  localhost:8080 → remote-db:5432                              │
│                                                                │
│  REMOTE FORWARD (-R):                                         │
│  Expose local service to remote                               │
│  ┌────────┐  :3000  ──SSH──  ┌────────┐  :8080               │
│  │ Laptop │──────────────────│ Server │──→ :8080 accessible  │
│  └────────┘                  └────────┘                       │
│  localhost:3000 accessible as server:8080                     │
│                                                                │
│  DYNAMIC FORWARD (-D):                                        │
│  SOCKS proxy through SSH                                      │
│  ┌────────┐  :1080  ──SSH──  ┌────────┐  → Internet          │
│  │ Laptop │──────────────────│ Server │──→ (any destination) │
│  └────────┘                  └────────┘                       │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Local port forwarding
# Access remote PostgreSQL through local port
ssh -L 5433:db-server:5432 user@bastion
# Now: psql -h localhost -p 5433

# Access remote web app
ssh -L 8080:internal-app:80 user@bastion
# Now: http://localhost:8080

# Remote port forwarding
# Expose local dev server to remote
ssh -R 8080:localhost:3000 user@server
# Now: server:8080 reaches your localhost:3000

# Dynamic SOCKS proxy
ssh -D 1080 user@server
# Configure browser to use SOCKS5 proxy localhost:1080

# Background tunnel
ssh -fN -L 5433:db:5432 user@bastion
# -f = background    -N = no command
```

---

## 6.5 SCP & SFTP — Secure File Transfer

```bash
# SCP — Secure Copy
scp file.txt user@server:/tmp/              # Upload
scp user@server:/var/log/app.log ./         # Download
scp -r ./deploy/ user@server:/opt/app/     # Recursive
scp -P 2222 file.txt user@server:/tmp/     # Custom port

# SFTP — SSH File Transfer Protocol
sftp user@server
sftp> ls                        # List remote files
sftp> cd /var/log               # Change remote directory
sftp> lcd /tmp                  # Change local directory
sftp> get app.log               # Download
sftp> put config.yaml           # Upload
sftp> mget *.log                # Download multiple
sftp> quit

# rsync over SSH (preferred for large transfers)
rsync -avz -e ssh ./app/ user@server:/opt/app/
rsync -avz --delete ./deploy/ user@server:/opt/deploy/
rsync -avz --progress large-file.tar.gz user@server:/tmp/
```

---

## 6.6 SSH Security Hardening

```bash
# /etc/ssh/sshd_config — Server configuration

# Disable root login
PermitRootLogin no

# Disable password authentication (key-only)
PasswordAuthentication no
PubkeyAuthentication yes

# Change default port (optional, security through obscurity)
Port 2222

# Limit users/groups
AllowUsers deploy admin
AllowGroups ssh-users

# Idle timeout
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable X11 forwarding
X11Forwarding no

# Limit authentication attempts
MaxAuthTries 3
MaxSessions 5

# Use only protocol 2
Protocol 2

# Restrict to specific addresses
ListenAddress 10.0.0.5
```

```bash
# After editing sshd_config
sudo sshd -t                    # Test config syntax
sudo systemctl restart sshd     # Apply changes

# Fail2ban — block brute-force SSH attacks
sudo apt install fail2ban
sudo systemctl enable --now fail2ban

# /etc/fail2ban/jail.local
# [sshd]
# enabled = true
# port = ssh
# maxretry = 3
# bantime = 3600
```

---

## 6.7 SSH Agent

```bash
# Start SSH agent
eval $(ssh-agent)

# Add key to agent
ssh-add ~/.ssh/id_ed25519
ssh-add -l                       # List loaded keys

# Agent forwarding (use local keys on remote server)
ssh -A user@bastion
# Now on bastion, you can SSH to other servers
# using your local keys (without copying them)

# In ~/.ssh/config
Host bastion
    ForwardAgent yes
```

---

## Real-World Scenario: CI/CD SSH Deployment

```bash
#!/bin/bash
# deploy.sh — CI/CD pipeline deploys via SSH

SERVER="deploy@prod-web"
APP_DIR="/opt/myapp"

echo "=== Deploying to production ==="

# 1. Upload new build
rsync -avz --delete \
    --exclude '.git' \
    --exclude 'node_modules' \
    ./dist/ ${SERVER}:${APP_DIR}/

# 2. Install dependencies and restart
ssh ${SERVER} << 'REMOTE'
    cd /opt/myapp
    npm install --production
    sudo systemctl restart myapp
    sleep 3
    systemctl is-active myapp && echo "Deploy SUCCESS" || echo "Deploy FAILED"
REMOTE
```

---

## Troubleshooting Tips

| Problem | Diagnosis | Solution |
|---------|-----------|----------|
| `Permission denied (publickey)` | Key not in authorized_keys | `ssh-copy-id user@server` |
| `WARNING: UNPROTECTED PRIVATE KEY` | Key file permissions too open | `chmod 600 ~/.ssh/id_ed25519` |
| `Host key verification failed` | Server key changed (or MITM) | Remove old key: `ssh-keygen -R hostname` |
| Connection timeout | Firewall, wrong IP/port | Check port 22 open, verify IP |
| `Too many authentication failures` | Too many keys tried | Use `-i keyfile` or config IdentityFile |
| SSH connection drops | Network idle timeout | Set `ServerAliveInterval 60` in config |

---

## Summary Table

| Command | Purpose | Example |
|---------|---------|---------|
| `ssh` | Remote shell | `ssh user@server` |
| `ssh-keygen` | Generate keys | `ssh-keygen -t ed25519` |
| `ssh-copy-id` | Copy public key | `ssh-copy-id user@server` |
| `scp` | Secure copy | `scp file user@server:/path` |
| `sftp` | Interactive transfer | `sftp user@server` |
| `ssh -L` | Local tunnel | `ssh -L 8080:remote:80 bastion` |
| `ssh -R` | Remote tunnel | `ssh -R 8080:localhost:3000 server` |
| `ssh-agent` | Key management | `eval $(ssh-agent); ssh-add` |

---

## Quick Revision Questions

1. **What is the difference between SSH password and key-based authentication? Why is key-based preferred?**
2. **What permissions must `~/.ssh/` and private key files have? Why?**
3. **How does SSH local port forwarding (`-L`) work? Give a use case.**
4. **What is a bastion/jump host and how do you configure ProxyJump?**
5. **Name three SSH hardening settings you would apply to a production server.**
6. **How would you set up SSH for a CI/CD pipeline to deploy to servers without interactive password entry?**

---

[← Previous: Firewall Configuration](05-firewall-configuration.md) | [Back to README](../README.md) | [Next: Network Troubleshooting →](07-network-troubleshooting.md)

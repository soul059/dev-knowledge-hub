# Chapter 6: FTP/SFTP

## Overview

FTP (File Transfer Protocol) and SFTP (SSH File Transfer Protocol) are used for transferring files between systems. While FTP is legacy and insecure, SFTP provides encrypted file transfer over SSH. DevOps engineers encounter these when managing deployments, configuration files, and data transfers — though modern alternatives like SCP, rsync, and object storage are often preferred.

---

## 6.1 FTP vs SFTP vs SCP vs rsync

```
┌──────────────────────────────────────────────────────────────┐
│          FILE TRANSFER PROTOCOL COMPARISON                   │
├──────────┬────────────┬────────────┬────────────┬────────────┤
│ Feature  │  FTP       │  SFTP      │  SCP       │  rsync     │
├──────────┼────────────┼────────────┼────────────┼────────────┤
│ Encrypted│  No*       │  Yes (SSH) │  Yes (SSH) │  Yes (SSH) │
│ Port     │  21 (ctrl) │  22        │  22        │  22/873    │
│          │  20 (data) │            │            │            │
│ Resume   │  Yes       │  Yes       │  No        │  Yes       │
│ Delta    │  No        │  No        │  No        │  Yes!      │
│ Dir list │  Yes       │  Yes       │  No        │  Yes       │
│ Interactive│ Yes      │  Yes       │  No        │  No        │
│ Speed    │  Fast      │  Moderate  │  Fast      │  Fastest** │
│ Security │  Insecure  │  Secure    │  Secure    │  Secure    │
│ Use in   │  Avoid     │  Good      │  Simple    │  Best for  │
│ DevOps   │            │            │  transfers │  large/sync│
└──────────┴────────────┴────────────┴────────────┴────────────┘

* FTPS (FTP over SSL) adds encryption but is still complex
** rsync only transfers changed portions of files (delta)
```

---

## 6.2 FTP Architecture

```
┌──────────────────────────────────────────────────────────────┐
│              FTP CONNECTION MODEL                            │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FTP uses TWO separate connections:                          │
│                                                              │
│  ┌──────────┐                         ┌──────────┐          │
│  │  Client  │─── Control (port 21) ──▶│  Server  │          │
│  │          │    Commands: USER, PASS, │          │          │
│  │          │    LIST, GET, PUT, QUIT  │          │          │
│  │          │                         │          │          │
│  │          │─── Data (port 20) ─────▶│          │          │
│  │          │    Actual file content   │          │          │
│  └──────────┘                         └──────────┘          │
│                                                              │
│  Active Mode:    Server connects TO client (firewall issues) │
│  Passive Mode:   Client connects TO server (firewall-friendly)│
│                                                              │
│  ⚠️  FTP sends passwords in PLAIN TEXT — NEVER use for       │
│     production or sensitive data!                            │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 6.3 SFTP (Recommended)

SFTP runs entirely over SSH (single connection, port 22). It is **not** FTP-over-SSH — it's a completely different protocol.

```
┌──────────────────────────────────────────────────────────────┐
│              SFTP vs FTP                                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  FTP:                                                        │
│  ┌────────┐   Port 21 (control)    ┌────────┐               │
│  │ Client │───────────────────────▶│ Server │               │
│  │        │   Port 20 (data)       │        │               │
│  │        │───────────────────────▶│        │               │
│  └────────┘   🔓 UNENCRYPTED       └────────┘               │
│                                                              │
│  SFTP:                                                       │
│  ┌────────┐   Port 22 (SSH)        ┌────────┐               │
│  │ Client │═══════════════════════▶│ Server │               │
│  └────────┘   🔒 ENCRYPTED         └────────┘               │
│               Single connection, all data secure             │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### SFTP Commands

```bash
# Connect to SFTP server
sftp user@server

# Interactive SFTP session
sftp> pwd                     # Remote working directory
sftp> lpwd                    # Local working directory
sftp> ls                      # List remote files
sftp> lls                     # List local files
sftp> cd /var/www             # Change remote directory
sftp> lcd ~/uploads           # Change local directory
sftp> get remote-file.txt     # Download file
sftp> put local-file.txt      # Upload file
sftp> get -r remote-dir/      # Download directory
sftp> put -r local-dir/       # Upload directory
sftp> mkdir new-directory     # Create remote directory
sftp> rm unwanted-file        # Delete remote file
sftp> exit                    # Close connection

# Non-interactive SFTP (batch mode)
sftp user@server <<EOF
cd /var/www/html
put index.html
put style.css
exit
EOF

# Using SSH key
sftp -i ~/.ssh/key.pem user@server
```

---

## 6.4 Modern Alternatives (DevOps Preferred)

### rsync (Best for Syncing)

```bash
# Sync local directory to remote (only changed files)
rsync -avz -e ssh ./project/ user@server:/opt/project/

# Flags:
# -a  archive mode (preserves permissions, timestamps)
# -v  verbose
# -z  compress during transfer
# -e  specify SSH
# --delete  remove files on destination not in source
# --dry-run  preview without making changes

# Sync with delete (mirror)
rsync -avz --delete -e ssh ./deploy/ user@server:/var/www/html/

# Exclude patterns
rsync -avz --exclude='node_modules' --exclude='.git' \
  -e ssh ./app/ user@server:/opt/app/

# Bandwidth limit (in KB/s)
rsync -avz --bwlimit=1000 -e ssh ./data/ user@server:/backup/
```

### SCP (Simple Copy)

```bash
# Upload file
scp file.txt user@server:/tmp/

# Download file
scp user@server:/var/log/app.log ./

# Copy directory
scp -r ./project/ user@server:/opt/

# With specific key
scp -i ~/.ssh/key.pem file.txt user@server:/tmp/

# Through bastion/jump host
scp -o ProxyJump=bastion file.txt user@target:/tmp/
```

### AWS S3 (Object Storage)

```bash
# Upload to S3
aws s3 cp file.txt s3://my-bucket/

# Sync directory to S3
aws s3 sync ./build/ s3://my-bucket/website/

# Download from S3
aws s3 cp s3://my-bucket/config.yml ./
```

---

## 6.5 CI/CD File Transfer Scenarios

### [~] Scenario: Deploying Build Artifacts

```
┌──────────────────────────────────────────────────────────────┐
│           CI/CD DEPLOYMENT FILE TRANSFER                     │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Option 1: rsync (SSH-based, delta transfer)                 │
│  ┌──────┐  rsync   ┌──────────┐                             │
│  │ CI   │─────────▶│  Server  │                             │
│  │Runner│  (SSH)    │/var/www/ │                             │
│  └──────┘           └──────────┘                             │
│                                                              │
│  Option 2: S3 as intermediary                                │
│  ┌──────┐  upload  ┌──────┐  download  ┌──────────┐         │
│  │ CI   │────────▶│  S3  │───────────▶│  Server  │         │
│  │Runner│          │Bucket│            │          │         │
│  └──────┘          └──────┘            └──────────┘         │
│                                                              │
│  Option 3: Container registry (modern)                       │
│  ┌──────┐  push    ┌──────┐  pull      ┌──────────┐         │
│  │ CI   │────────▶│ ECR/ │───────────▶│  K8s/ECS │         │
│  │Runner│          │Docker│            │  Cluster │         │
│  └──────┘          │ Hub  │            └──────────┘         │
│                    └──────┘                                  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

```yaml
# GitHub Actions: Deploy via rsync
- name: Deploy to server
  uses: burnett01/rsync-deployments@5.2
  with:
    switches: -avz --delete
    path: dist/
    remote_path: /var/www/html/
    remote_host: ${{ secrets.DEPLOY_HOST }}
    remote_user: deploy
    remote_key: ${{ secrets.SSH_PRIVATE_KEY }}
```

---

## 6.6 Troubleshooting Tips

| Problem | Cause | Solution |
|---------|-------|----------|
| SFTP "Connection refused" | SSH not running or port blocked | Check sshd, security group port 22 |
| "Permission denied" | Wrong key or file permissions | Check key, chmod 600, check user |
| FTP passive mode fails | Firewall blocking data ports | Use SFTP instead, or open passive range |
| Slow transfer | No compression, large files | Use rsync -z, or split large files |
| rsync "permission denied" | Remote user lacks write access | Check directory ownership, use sudo |
| Transfer interrupted | Network instability | Use rsync (supports resume) |

---

## Summary Table

| Concept | Key Points |
|---------|------------|
| FTP | Legacy, insecure (plain text), port 21/20 — avoid in production |
| FTPS | FTP + SSL/TLS — complex, still avoid if possible |
| SFTP | SSH-based file transfer, port 22, encrypted — recommended |
| SCP | Simple SSH copy, no resume, good for quick transfers |
| rsync | Delta sync over SSH, compression, resume — best for deployments |
| S3/Blob | Cloud object storage — preferred for CI/CD artifact storage |
| Best practice | Use rsync or object storage; never use plain FTP |

---

## Quick Revision Questions

1. **Why should FTP never be used for production file transfers?**
2. **What is the key difference between SFTP and FTPS?**
3. **When would you choose rsync over SCP? Give a specific use case.**
4. **Write the rsync command to sync a project directory to a server, excluding node_modules.**
5. **In a CI/CD pipeline, what are three methods to deploy build artifacts to a server?**
6. **What makes rsync efficient for large, frequently updated directories?**

---

## Navigation

| Previous | Up | Next |
|----------|-----|------|
| [← SSH](05-ssh.md) | [README](../README.md) | [VLANs →](../03-Network-Architecture/01-vlans.md) |

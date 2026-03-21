# FTP/SFTP/SCP

## Unit 5 - Topic 4 | Application Layer Protocols

---

## Overview

**FTP (File Transfer Protocol)** is one of the oldest internet protocols for file transfer, but it sends everything — including credentials — in **cleartext**. **SFTP** and **SCP** are secure alternatives using SSH encryption. Understanding FTP's weaknesses is essential for both exploitation and defense.

---

## 1. Protocol Comparison

| Feature | FTP | FTPS | SFTP | SCP |
|---------|-----|------|------|-----|
| **Port** | 20/21 | 990 (implicit) | 22 | 22 |
| **Encryption** | ❌ None | ✅ TLS/SSL | ✅ SSH | ✅ SSH |
| **Authentication** | Cleartext | TLS-protected | SSH keys/password | SSH keys/password |
| **Based On** | FTP protocol | FTP + TLS | SSH subsystem | SSH protocol |
| **Firewall Friendly** | ❌ (active mode issues) | ⚠️ | ✅ (single port) | ✅ (single port) |
| **Recommendation** | ❌ Never use | ⚠️ Acceptable | ✅ Preferred | ✅ Preferred |

---

## 2. FTP Security Issues

| Vulnerability | Description |
|--------------|-------------|
| **Cleartext Credentials** | Username and password sent unencrypted |
| **Cleartext Data** | File contents transmitted in plaintext |
| **Anonymous FTP** | Guest access may expose sensitive files |
| **Bounce Attack** | Abuse FTP PORT command to scan other hosts |
| **Brute Force** | No default lockout mechanism |
| **Directory Traversal** | Misconfiguration may allow access outside FTP root |

```
FTP Credential Capture (Wireshark):
┌────────────────────────────────┐
│ 220 FTP server ready           │
│ USER admin                     │ ← Username in cleartext!
│ 331 Password required          │
│ PASS S3cretP@ss!              │ ← Password in cleartext!
│ 230 Login successful           │
└────────────────────────────────┘

An attacker with network access (ARP spoof/sniff) 
can capture these credentials trivially.
```

---

## 3. FTP Reconnaissance & Attack

```bash
# Banner grabbing
nc -v target.com 21
nmap -sV -p 21 target.com

# Check for anonymous login
ftp target.com
> Username: anonymous
> Password: (blank or email)

# Nmap FTP scripts
nmap -p 21 --script ftp-anon target.com          # Anonymous check
nmap -p 21 --script ftp-brute target.com          # Brute force
nmap -p 21 --script ftp-bounce target.com         # Bounce attack
nmap -p 21 --script ftp-vuln-cve2010-4221 target  # ProFTPD exploit

# Brute force with Hydra
hydra -l admin -P /usr/share/wordlists/rockyou.txt ftp://target.com
```

---

## 4. Secure Alternatives

### SFTP (SSH File Transfer Protocol)

```bash
# SFTP — Runs over SSH (port 22)
sftp user@server.com
sftp> put localfile.txt              # Upload
sftp> get remotefile.txt             # Download
sftp> ls                             # List files

# Key-based authentication (most secure)
sftp -i ~/.ssh/id_ed25519 user@server.com
```

### SCP (Secure Copy Protocol)

```bash
# SCP — Copy files over SSH
scp localfile.txt user@server:/path/          # Upload
scp user@server:/path/file.txt ./             # Download
scp -r folder/ user@server:/path/             # Copy directory
scp -i ~/.ssh/id_ed25519 file user@server:/   # With SSH key
```

---

## 5. FTP Hardening (If Must Use FTP)

| Practice | Description |
|----------|-------------|
| **Use SFTP instead** | Always prefer SFTP over FTP |
| **Disable anonymous** | Remove anonymous access |
| **Chroot jail** | Restrict users to their home directory |
| **TLS (FTPS)** | Enable TLS if FTP is required |
| **IP restrictions** | Whitelist allowed source IPs |
| **Disable PORT command** | Prevent bounce attacks |
| **Monitor logs** | Watch for brute force and anomalous access |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **FTP** | Cleartext protocol — credentials and data exposed |
| **SFTP** | Secure — runs over SSH (port 22) |
| **SCP** | Secure — file copy over SSH |
| **Anonymous FTP** | Major security risk if misconfigured |
| **Best Practice** | Replace FTP with SFTP everywhere |

---

## Quick Revision Questions

1. **Why is FTP considered insecure?**
2. **What is the difference between SFTP and FTPS?**
3. **What can an attacker capture from FTP traffic?**
4. **What is an FTP bounce attack?**
5. **How do you check for anonymous FTP access with Nmap?**
6. **Why is SFTP preferred over SCP for interactive file management?**

---

[← Previous: SMTP/IMAP/POP3](03-smtp-imap-pop3.md) | [Next: SSH →](05-ssh.md)

---

*Unit 5 - Topic 4 of 6 | [Back to README](../README.md)*

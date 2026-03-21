# Hash Cracking Basics

## Unit 4 - Topic 6 | Password Attacks

---

## Overview

**Hash cracking** recovers plaintext passwords from their hash representations. When you dump password databases, capture NTLMv2 hashes via Responder, or extract hashes from `/etc/shadow`, hash cracking is how you turn those hashes into usable credentials.

---

## 1. Common Hash Types

| Hash Type | Example | Hashcat Mode | Use Case |
|-----------|---------|:------------:|----------|
| **MD5** | `5f4dcc3b5aa765d61d8327deb882cf99` | 0 | Web apps, legacy |
| **SHA-1** | `5baa61e4c9b93f3f0682...` | 100 | Legacy authentication |
| **SHA-256** | `5e884898da28047151d0...` | 1400 | Modern apps |
| **NTLM** | `a4f49c406510bdcab688...` | 1000 | Windows passwords |
| **NTLMv2** | `admin::CORP:1122...` | 5600 | Responder captures |
| **bcrypt** | `$2b$12$LJ3m4...` | 3200 | Modern web apps |
| **SHA-512crypt** | `$6$rounds=5000$...` | 1800 | Linux /etc/shadow |
| **Kerberos TGS** | `$krb5tgs$23$*...` | 13100 | Kerberoasting |
| **Net-NTLMv2** | `user::domain:...` | 5600 | Network capture |

---

## 2. Hash Identification

```bash
# === HASH-IDENTIFIER ===
hash-identifier
# Paste hash → identifies type

# === HASHID ===
hashid "5f4dcc3b5aa765d61d8327deb882cf99"
# [+] MD5
# [+] MD4
# [+] Double MD5

# === MANUAL IDENTIFICATION ===
# 32 hex chars        → MD5
# 40 hex chars        → SHA-1
# 64 hex chars        → SHA-256
# 128 hex chars       → SHA-512
# 32 hex chars        → NTLM (same length as MD5!)
# $2b$12$...          → bcrypt
# $6$...              → SHA-512crypt (Linux)
# $1$...              → MD5crypt (Linux)
# $krb5tgs$...        → Kerberos TGS

# === HASHCAT EXAMPLE HASHES ===
# hashcat.net/wiki/doku.php?id=example_hashes
# Reference for all hash formats
```

---

## 3. Cracking with Hashcat

```bash
# === BASIC USAGE ===
hashcat -m [MODE] -a [ATTACK] [HASH_FILE] [WORDLIST]

# Dictionary attack on NTLM:
hashcat -m 1000 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# Dictionary + rules:
hashcat -m 1000 -a 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Brute force (mask):
hashcat -m 1000 -a 3 hashes.txt ?u?l?l?l?l?d?d?s

# Combination attack:
hashcat -m 1000 -a 1 hashes.txt words1.txt words2.txt
# Combines words from both lists

# === SHOW CRACKED ===
hashcat -m 1000 hashes.txt --show

# === BENCHMARK ===
hashcat -b   # Shows hash speeds for your GPU

# === USEFUL FLAGS ===
# -o output.txt     → Save cracked to file
# --force           → Override warnings
# --username         → Hash file includes usernames
# -w 3              → Workload profile (1=low, 3=high)
# --potfile-disable  → Don't use potfile
```

---

## 4. Cracking with John the Ripper

```bash
# === BASIC USAGE ===
john hashes.txt
# Auto-detects hash type and cracks

# Specify format:
john --format=raw-md5 hashes.txt
john --format=NT hashes.txt
john --format=sha512crypt hashes.txt

# With wordlist:
john --wordlist=rockyou.txt hashes.txt

# With rules:
john --wordlist=rockyou.txt --rules=best64 hashes.txt

# === SHOW CRACKED ===
john --show hashes.txt

# === HASH EXTRACTION TOOLS ===
# Linux shadow file:
unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt

# Windows SAM:
# Use secretsdump.py, mimikatz, or reg save

# ZIP files:
zip2john encrypted.zip > zip_hash.txt
john zip_hash.txt

# PDF files:
pdf2john document.pdf > pdf_hash.txt
john pdf_hash.txt
```

---

## 5. Cracking Speed Comparison

| Hash Type | Hashcat Speed (RTX 4090) | Difficulty |
|-----------|:------------------------:|:----------:|
| **MD5** | ~164 GH/s | Very easy |
| **NTLM** | ~300 GH/s | Very easy |
| **SHA-1** | ~28 GH/s | Easy |
| **SHA-256** | ~11 GH/s | Moderate |
| **NTLMv2** | ~12 GH/s | Moderate |
| **bcrypt** | ~184 KH/s | Hard |
| **Argon2** | ~3 KH/s | Very hard |
| **scrypt** | ~2 MH/s | Hard |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **Hashcat** | GPU-based, fastest cracker |
| **John** | CPU-based, versatile formats |
| **Hash mode** | `-m` specifies hash type in Hashcat |
| **Rules** | Modify wordlist entries for variations |
| **Hash ID** | hashid / hash-identifier for type detection |
| **Speed** | NTLM/MD5 fast; bcrypt/Argon2 very slow |

---

## Quick Revision Questions

1. **How do you identify an unknown hash type?**
2. **What is the difference between Hashcat and John?**
3. **What Hashcat mode is used for NTLM hashes?**
4. **Why is bcrypt harder to crack than MD5?**
5. **How do you extract hashes from a Linux shadow file?**

---

[← Previous: Credential Stuffing](05-credential-stuffing.md) | [Next: Password Attack Tools →](07-password-attack-tools.md)

---

*Unit 4 - Topic 6 of 7 | [Back to README](../README.md)*

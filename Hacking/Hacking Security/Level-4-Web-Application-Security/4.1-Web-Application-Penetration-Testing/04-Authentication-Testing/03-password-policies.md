# Password Policies

## Unit 4: Authentication Testing — Topic 3

## 🎯 Overview

Password policies define the rules for password creation, storage, and management. Weak policies lead to easily guessable passwords, while overly restrictive policies drive users to unsafe workarounds. Penetration testers assess both the strength of password policies and their server-side enforcement to identify authentication weaknesses.

---

## 1. Password Policy Assessment

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD POLICY TESTING MATRIX                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Test                          │  Expected  │  Result        │
│  ─────────────────────────────┼────────────┼──────────────  │
│  Minimum length enforcement    │  8+ chars  │  Test: "abc"   │
│  Maximum length limit          │  64+ chars │  Test: 200 A's │
│  Uppercase required            │  Yes       │  Test: alllower│
│  Lowercase required            │  Yes       │  Test: ALLUPPER│
│  Number required               │  Yes       │  Test: NoNums  │
│  Special char required         │  Yes       │  Test: NoSpec  │
│  Common password block         │  Yes       │  Test: password│
│  Breached password check       │  Yes       │  Test: known   │
│  Username in password blocked  │  Yes       │  Test: admin123│
│  Password history enforced     │  Yes       │  Reuse old pass│
│  Server-side enforcement       │  Yes       │  Bypass client │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Testing Password Strength Requirements

```bash
# Test minimum length (bypass client-side validation)
curl -X POST https://target.com/register \
  -d "username=test1&password=abc"          # 3 chars
curl -X POST https://target.com/register \
  -d "username=test2&password=abcdefg"      # 7 chars

# Test maximum length (potential truncation or DoS)
python3 -c "print('A'*10000)" | \
  xargs -I {} curl -X POST https://target.com/register \
  -d "username=test3&password={}"

# Test common passwords
for pass in password 123456 admin qwerty letmein; do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    -X POST https://target.com/register \
    -d "username=testuser&password=$pass")
  echo "$pass → $code"
done

# Test password same as username
curl -X POST https://target.com/register \
  -d "username=testadmin&password=testadmin"
```

---

## 3. Password Storage Assessment

```
┌──────────────────────────────────────────────────────────────┐
│              PASSWORD HASHING COMPARISON                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Method          │  Security  │  Example                     │
│  ────────────────┼────────────┼────────────────────────────  │
│  Plaintext       │  ❌ None   │  password123                 │
│  Base64          │  ❌ None   │  cGFzc3dvcmQxMjM=            │
│  MD5             │  ❌ Weak   │  482c811da5d5b4bc...          │
│  SHA-1           │  ❌ Weak   │  cbfdac6008f9cab4...          │
│  SHA-256         │  ⚠ Fair    │  ef92b778bafe77...            │
│  SHA-256 + Salt  │  ⚠ Fair    │  salt + SHA-256(salt+pass)   │
│  bcrypt          │  ✅ Strong  │  $2b$12$LJ3...               │
│  scrypt          │  ✅ Strong  │  Memory-hard function        │
│  Argon2          │  ✅ Best    │  $argon2id$v=19...           │
│  PBKDF2          │  ✅ Good    │  iterations * HMAC-SHA256    │
└──────────────────────────────────────────────────────────────┘
```

```bash
# If password hashes are obtained, identify the hash type
hashid '$2b$12$LJ3m4ys'           # bcrypt
hashid '482c811da5d5b4bc...'      # MD5
hashid 'cbfdac6008f9cab4...'      # SHA-1

# Crack weak hashes
hashcat -m 0 md5_hashes.txt rockyou.txt     # MD5
hashcat -m 100 sha1_hashes.txt rockyou.txt  # SHA-1
hashcat -m 1400 sha256_hashes.txt rockyou.txt  # SHA-256
hashcat -m 3200 bcrypt_hashes.txt rockyou.txt  # bcrypt (slow)

# John the Ripper
john --format=bcrypt hashes.txt --wordlist=rockyou.txt
```

---

## 4. Client-Side vs Server-Side Enforcement

```
┌──────────────────────────────────────────────────────────────┐
│              VALIDATION BYPASS                                │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client-Side (JavaScript):                                   │
│  if (password.length < 8) alert("Too short!");              │
│  → Easily bypassed by disabling JS or using Burp Suite      │
│                                                              │
│  Server-Side (Backend):                                      │
│  if len(password) < 8:                                       │
│      return error("Password must be 8+ characters")          │
│  → Cannot be bypassed by the client                          │
│                                                              │
│  ALWAYS verify server-side enforcement:                      │
│  1. Intercept registration request in Burp                   │
│  2. Modify password to weak value ("a")                      │
│  3. Forward request                                          │
│  4. If account created → server doesn't enforce!             │
└──────────────────────────────────────────────────────────────┘
```

---

## 5. Password Policy Best Practices (NIST 800-63B)

| Guideline | NIST Recommendation | Common Practice |
|-----------|-------------------|-----------------|
| Minimum length | 8 characters | 8-12 characters |
| Maximum length | At least 64 characters | Often 20-30 (too low) |
| Complexity rules | NOT recommended | Still widely used |
| Password expiration | NOT recommended | Often 90 days |
| Breached password check | Required | Rarely implemented |
| Password hints | NOT allowed | Sometimes present |
| Knowledge questions | NOT recommended | Still used |
| Paste in password field | MUST be allowed | Often blocked |

```bash
# Test NIST compliance issues:
# 1. Can you paste passwords? (blocking paste = poor practice)
# 2. Is there forced rotation? (unnecessary if strong)
# 3. Are complexity rules the only check? (no breach check)
# 4. Is max length too low? (breaks password managers)
# 5. Are password hints stored? (information leak)
```

---

## 6. Account Lockout Policy Testing

```bash
# Test lockout threshold
for i in $(seq 1 20); do
  code=$(curl -s -o /dev/null -w "%{http_code}" \
    -X POST https://target.com/login \
    -d "username=admin&password=wrong$i")
  echo "Attempt $i: $code"
done

# Check lockout behavior:
# • After how many attempts? (3? 5? 10?)
# • Duration? (5 min? 30 min? permanent?)
# • Is it IP-based or account-based?
# • Does it apply to API endpoints too?
# • Can lockout be weaponized? (DoS via account lockout)

# Test lockout bypass:
# • Different IP (X-Forwarded-For)
# • Different endpoint (/api/login vs /login)
# • Different case (Admin vs admin)
# • After lockout, does valid login work?
```

---

## 📊 Summary Table

| Policy Element | Weak | Strong |
|---------------|------|--------|
| Min Length | 4-6 chars | 12+ chars |
| Max Length | 16 chars | 64+ chars |
| Complexity | None or basic | Breach check + length |
| Storage | MD5/SHA1 | bcrypt/Argon2 |
| Lockout | None | 5 attempts, 15 min |
| Enforcement | Client-side only | Server-side |
| Rotation | Every 30 days | On compromise only |

---

## ❓ Revision Questions

1. Why does NIST no longer recommend password complexity requirements?
2. How do you test whether password policies are enforced server-side?
3. What is the difference between bcrypt and MD5 for password storage?
4. How can account lockout policies be weaponized by attackers?
5. Why should password pasting be allowed in login fields?
6. What is a breached password check and why is it recommended?

---

*Previous: [02-brute-force-attacks.md](02-brute-force-attacks.md) | Next: [04-mfa-testing.md](04-mfa-testing.md)*

---

*[Back to README](../README.md)*

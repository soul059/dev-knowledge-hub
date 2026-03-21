# Email Verification

## Unit 6 - Topic 2 | Email Reconnaissance

---

## Overview

**Email verification** confirms whether harvested email addresses are valid and active. Verifying emails before use increases the success rate of social engineering campaigns and reduces detection from bounced messages.

---

## 1. Verification Methods

```
EMAIL VERIFICATION TECHNIQUES:

METHOD 1: SMTP VRFY Command
├── Connect to target mail server
├── Use VRFY or RCPT TO command
├── Server responds: valid or invalid
└── ⚠️ Many servers disable VRFY

METHOD 2: SMTP RCPT TO
├── More reliable than VRFY
├── Simulate sending an email
├── Check if recipient is accepted
└── Don't actually send the message

METHOD 3: Third-Party Services
├── hunter.io/email-verifier
├── emailhippo.com
├── neverbounce.com
└── API-based bulk verification

METHOD 4: Catch-All Detection
├── Some domains accept ALL addresses
├── user123random@target.com → 250 OK
├── This means verification is unreliable
└── Must use other confirmation methods
```

---

## 2. SMTP Verification Techniques

```bash
# === MANUAL SMTP VERIFICATION ===
# Step 1: Find MX server
dig target.com MX +short
# 10 mail.target.com

# Step 2: Connect to mail server
telnet mail.target.com 25

# Step 3: SMTP handshake
HELO test.com
MAIL FROM: <test@test.com>
RCPT TO: <john.smith@target.com>

# RESPONSES:
# 250 OK          → Email EXISTS ✅
# 550 User unknown → Email INVALID ❌
# 252 Cannot VRFY  → Server won't confirm

# Step 4: Quit without sending
QUIT

# === AUTOMATED SMTP VERIFICATION ===
# smtp-user-enum (Kali tool)
smtp-user-enum -M RCPT -u john.smith -t mail.target.com

# Enumerate from list:
smtp-user-enum -M RCPT -U userlist.txt -t mail.target.com

# === NMAP SMTP ENUM ===
nmap --script smtp-enum-users -p 25 mail.target.com
```

---

## 3. Verification Response Codes

| SMTP Code | Meaning | Interpretation |
|:---------:|---------|:---:|
| **250** | OK / User exists | ✅ Valid |
| **251** | User not local, will forward | ✅ Valid |
| **252** | Cannot VRFY but will accept | ⚠️ Maybe |
| **450** | Mailbox unavailable (temp) | ⚠️ Try later |
| **550** | User unknown | ❌ Invalid |
| **551** | User not local | ❌ Invalid |
| **553** | Mailbox name not allowed | ❌ Invalid |

---

## 4. Bulk Verification Workflow

```bash
# === STEP 1: PREPARE EMAIL LIST ===
cat emails.txt
# john.smith@target.com
# jane.doe@target.com
# admin@target.com
# bob.fake@target.com

# === STEP 2: CHECK FOR CATCH-ALL ===
# Test with obviously fake email:
# randomstring123456@target.com
# If accepted → catch-all domain (verification unreliable)

# === STEP 3: VERIFY EACH EMAIL ===
# Using smtp-user-enum:
smtp-user-enum -M RCPT -U emails.txt -D target.com -t mail.target.com

# === STEP 4: CATEGORIZE RESULTS ===
# verified_valid.txt    → 250 responses
# verified_invalid.txt  → 550 responses
# unverified.txt        → 252/timeout responses

# === STEP 5: RATE LIMITING ===
# ⚠️ Don't verify too fast!
# Rate limit: 1 query per 5-10 seconds
# Rotate source IPs if possible
# Use different HELO domains
```

---

## 5. OPSEC Considerations

| Risk | Mitigation |
|------|-----------|
| **IP blocking** | Use multiple source IPs |
| **Logging** | Target may log all SMTP connections |
| **Rate limiting** | Slow down queries (5-10 sec intervals) |
| **Detection** | Unusual SMTP activity triggers alerts |
| **Catch-all** | Test with random address first |
| **Greylisting** | Retry after delay if temporarily rejected |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SMTP VRFY** | Direct verification (often disabled) |
| **RCPT TO** | More reliable — check if recipient accepted |
| **250 OK** | Email exists and is valid |
| **550** | Email doesn't exist |
| **Catch-all** | Domain accepts everything — unreliable |
| **OPSEC** | Rate limit and watch for detection |

---

## Quick Revision Questions

1. **What SMTP commands verify email addresses?**
2. **What does a 250 response code mean?**
3. **What is a catch-all domain and how does it affect verification?**
4. **What tool automates SMTP user enumeration?**
5. **What OPSEC precautions should you take during verification?**

---

[← Previous: Email Harvesting](01-email-harvesting.md) | [Next: MX Record Analysis →](03-mx-record-analysis.md)

---

*Unit 6 - Topic 2 of 5 | [Back to README](../README.md)*

# Unit 3: Phishing — Topic 2: Email Phishing

## Overview

**Email phishing** is the most common form of phishing, using fraudulent emails to trick recipients into clicking malicious links, opening weaponized attachments, or providing sensitive information. Understanding how phishing emails are crafted, delivered, and detected is essential for both offensive testing and defensive security.

---

## 1. Anatomy of a Phishing Email

```
PHISHING EMAIL COMPONENTS:

FROM: support@micros0ft-security.com    ← Spoofed/lookalike sender
TO: victim@company.com                  ← Target
SUBJECT: [URGENT] Your account has been compromised!  ← Fear + Urgency

─────────────────────────────────────────────────
Dear Valued Customer,

We detected unusual activity on your account.     ← Fear trigger
Your access will be suspended within 24 hours     ← Urgency/Scarcity
unless you verify your identity immediately.

Please click below to verify your account:        ← Call to action

[Verify Now]                                      ← Malicious link
→ https://micros0ft-security.com/verify           ← Lookalike domain

If you did not request this, please contact        ← Fake legitimacy
our security team immediately.

Best regards,
Microsoft Security Team                           ← Authority

─────────────────────────────────────────────────
© 2024 Microsoft Corporation                      ← Fake branding
This is an automated notification.
Unsubscribe: [here]                              ← More fake legitimacy
```

---

## 2. Red Flags in Phishing Emails

```
INDICATORS OF PHISHING:

SENDER ANALYSIS:
  ✗ External email claiming to be internal
  ✗ Lookalike domain (micros0ft vs microsoft)
  ✗ Reply-to differs from sender
  ✗ Generic sender (support@, noreply@)
  ✗ Free email service (gmail, yahoo) for business

CONTENT ANALYSIS:
  ✗ Urgency language ("immediately", "within 24 hours")
  ✗ Fear language ("suspended", "compromised", "blocked")
  ✗ Generic greeting ("Dear Customer" vs your name)
  ✗ Grammar/spelling errors
  ✗ Mismatched branding or logo quality
  ✗ Threatening consequences

LINK ANALYSIS:
  ✗ Hover text differs from display text
  ✗ Shortened URLs (bit.ly, tinyurl)
  ✗ IP addresses instead of domain names
  ✗ Lookalike domains (paypa1.com, amaz0n.com)
  ✗ Extra subdomains (login.microsoft.com.evil.com)
  ✗ HTTP instead of HTTPS

ATTACHMENT ANALYSIS:
  ✗ Unexpected attachments
  ✗ Double extensions (invoice.pdf.exe)
  ✗ Macro-enabled documents (.docm, .xlsm)
  ✗ Archive files (.zip, .rar) containing executables
  ✗ HTML files (local phishing pages)
```

---

## 3. Email Spoofing Techniques

```
SPOOFING METHODS:

SIMPLE HEADER MANIPULATION:
  → Set From header to any email address
  → SMTP doesn't verify sender by default
  → Blocked by: SPF, DKIM, DMARC

LOOKALIKE DOMAINS:
  → Register similar domain: g00gle.com, goggle.com
  → IDN homograph: gооgle.com (Cyrillic 'о')
  → Typosquatting: gogle.com, googl.com
  → TLD variation: google.co, google.security

DISPLAY NAME SPOOFING:
  → From: "CEO Name" <attacker@gmail.com>
  → Email client shows "CEO Name" prominently
  → Users don't check the actual email address
  → Most effective on mobile (hides address)

COMPROMISED ACCOUNTS:
  → Use actual compromised email account
  → Emails pass all authentication checks
  → Most convincing and hardest to detect
  → SPF, DKIM, DMARC all pass!

LEGITIMATE SERVICES:
  → Send phishing via Google Docs share
  → Use SharePoint/OneDrive sharing
  → Abuse contact forms, calendar invites
  → Emails come FROM legitimate services
```

---

## 4. Phishing Email Templates

```
COMMON PHISHING PRETEXTS:

CREDENTIAL HARVESTING:
  → "Password expiring — reset now"
  → "Verify your email address"
  → "Unusual sign-in activity detected"
  → "Your mailbox is full — upgrade"

MALWARE DELIVERY:
  → "Invoice attached for your review"
  → "Shipping notification — track your package"
  → "HR: Updated benefits enrollment form"
  → "Voicemail from [phone number]"

INFORMATION GATHERING:
  → "IT survey — help us improve"
  → "Confirm your details for directory update"
  → "Tax form requires your SSN verification"

FINANCIAL:
  → "Wire transfer confirmation needed"
  → "Updated banking details — please note"
  → "Purchase order requires approval"
  → "Refund available — claim now"

CURRENT EVENTS (Timely):
  → Tax season: "IRS refund notification"
  → COVID: "Updated health policy — review"
  → Elections: "Voter registration confirmation"
  → Natural disaster: "Relief fund donation"
```

---

## Summary Table

| Component | Legitimate Email | Phishing Email |
|-----------|-----------------|----------------|
| Sender | Verified domain | Spoofed/lookalike |
| Greeting | Personalized | Generic |
| Tone | Professional | Urgent/threatening |
| Links | Match display text | Mismatch/obfuscated |
| Action | Reasonable request | Click/download/urgent |
| Errors | None/few | Often present |

---

## Revision Questions

1. What are the key components of a phishing email?
2. How do you identify red flags in a suspicious email?
3. What email spoofing techniques bypass basic checks?
4. What are the most effective phishing email pretexts?
5. How do SPF, DKIM, and DMARC help prevent email spoofing?

---

*Previous: [01-phishing-types.md](01-phishing-types.md) | Next: [03-spear-phishing.md](03-spear-phishing.md)*

---

*[Back to README](../README.md)*

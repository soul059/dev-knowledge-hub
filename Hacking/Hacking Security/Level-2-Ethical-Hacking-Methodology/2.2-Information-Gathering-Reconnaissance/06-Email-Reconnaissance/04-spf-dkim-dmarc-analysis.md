# SPF, DKIM, and DMARC Analysis

## Unit 6 - Topic 4 | Email Reconnaissance

---

## Overview

**SPF**, **DKIM**, and **DMARC** are email authentication mechanisms that prevent spoofing and phishing. Analyzing these records reveals the target's email security posture and determines whether email spoofing attacks are feasible.

---

## 1. Understanding the Three Protocols

```
EMAIL AUTHENTICATION STACK:

┌──────────────────────────────────────────────────────────┐
│  SPF (Sender Policy Framework)                           │
│  "Which servers are ALLOWED to send email for us?"       │
│  DNS TXT record: v=spf1 include:... -all                │
├──────────────────────────────────────────────────────────┤
│  DKIM (DomainKeys Identified Mail)                       │
│  "Is this email cryptographically SIGNED by us?"         │
│  DNS TXT record: selector._domainkey.target.com          │
├──────────────────────────────────────────────────────────┤
│  DMARC (Domain-based Message Authentication)             │
│  "What should receivers DO with failed SPF/DKIM?"        │
│  DNS TXT record: _dmarc.target.com                       │
└──────────────────────────────────────────────────────────┘

AUTHENTICATION FLOW:
Incoming email → Check SPF (IP allowed?)
               → Check DKIM (signature valid?)
               → Check DMARC (policy: reject/quarantine/none)
```

---

## 2. Querying Records

```bash
# === SPF RECORD ===
dig target.com TXT +short | grep "v=spf1"
# "v=spf1 include:_spf.google.com include:servers.mcsv.net ~all"

# SPF QUALIFIERS:
# +all  → Pass ALL (basically no protection!)
# ~all  → SoftFail (accept but mark suspicious)
# -all  → Fail (reject unauthorized senders) ← STRICTEST
# ?all  → Neutral (no policy)

# === DKIM RECORD ===
# Need to know the selector (common: google, default, s1, k1)
dig google._domainkey.target.com TXT +short
dig default._domainkey.target.com TXT +short
dig s1._domainkey.target.com TXT +short

# DKIM record contains public key for signature verification

# === DMARC RECORD ===
dig _dmarc.target.com TXT +short
# "v=DMARC1; p=reject; rua=mailto:dmarc@target.com; pct=100"

# DMARC POLICIES:
# p=none       → Monitor only (no enforcement) ← WEAKEST
# p=quarantine → Send to spam folder
# p=reject     → Reject entirely ← STRICTEST
```

---

## 3. Security Posture Assessment

| Configuration | Spoofing Risk | Assessment |
|:---:|:---:|:---:|
| No SPF + No DMARC | ★★★★★ Critical | Completely spoofable |
| SPF `~all` + No DMARC | ★★★★ High | Soft fail, still delivered |
| SPF `-all` + No DMARC | ★★★ Medium | Rejected by some, not all |
| SPF `-all` + DMARC `p=none` | ★★★ Medium | Monitored but not enforced |
| SPF `-all` + DMARC `p=quarantine` | ★★ Low | Goes to spam |
| SPF `-all` + DKIM + DMARC `p=reject` | ★ Very Low | Full protection |

```bash
# === AUTOMATED ANALYSIS ===
# mxtoolbox.com/SuperTool — comprehensive email config check
# dmarcian.com — DMARC analyzer
# mail-tester.com — send test email for score

# === SPOOFCHECK TOOL ===
spoofcheck target.com
# Output: "target.com appears to be spoofable!"
# Checks: SPF, DMARC, and identifies weaknesses
```

---

## 4. Exploiting Weak Configurations

```
SCENARIO 1: NO SPF, NO DMARC
─────────────────────────────
Attack: Send spoofed email as ceo@target.com
Result: Delivered to recipient's inbox
Risk: Extremely high for phishing success

SCENARIO 2: SPF SOFTFAIL (~all)
─────────────────────────────────
Attack: Send from unauthorized server
Result: Email marked but usually delivered
Risk: High — many servers ignore softfail

SCENARIO 3: DMARC p=none
─────────────────────────────────
Attack: Same as no DMARC for enforcement
Result: Reports sent to target, but email delivered
Risk: Medium — target gets reports but no blocking

SCENARIO 4: SPF -all + DMARC p=reject
─────────────────────────────────────
Attack: Spoofing blocked by most servers
Bypass: Register similar domain (targеt.com with Cyrillic 'е')
Risk: Low — must use alternative techniques
```

---

## 5. SPF Record Analysis

```bash
# === RECURSIVE SPF LOOKUP ===
# SPF records can include other domains
dig target.com TXT +short
# "v=spf1 include:_spf.google.com include:servers.mcsv.net ~all"

# Expand includes:
dig _spf.google.com TXT +short
# Shows all Google IP ranges allowed to send

# === SPF LOOKUP COUNT ===
# SPF has a 10-lookup limit
# If exceeds 10 → SPF breaks → PermError
# → Emails fail SPF check
# → Potential spoofing window!

# Check lookup count:
# dmarcanalyzer.com/spf/checker
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SPF** | Authorized sending IPs (-all = strict) |
| **DKIM** | Cryptographic email signatures |
| **DMARC** | Policy for failed checks (reject = strict) |
| **spoofcheck** | Test if domain is spoofable |
| **~all** | Soft fail — often still delivered |
| **p=none** | Monitor only — no enforcement |

---

## Quick Revision Questions

1. **What do SPF, DKIM, and DMARC each protect against?**
2. **What is the difference between ~all and -all in SPF?**
3. **What DMARC policy provides the strongest protection?**
4. **How do you check if a domain is spoofable?**
5. **What happens when SPF exceeds 10 DNS lookups?**

---

[← Previous: MX Record Analysis](03-mx-record-analysis.md) | [Next: Breached Data Sources →](05-breached-data-sources.md)

---

*Unit 6 - Topic 4 of 5 | [Back to README](../README.md)*

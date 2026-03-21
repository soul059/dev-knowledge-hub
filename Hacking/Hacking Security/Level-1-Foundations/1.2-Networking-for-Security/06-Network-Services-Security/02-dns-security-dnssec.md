# DNS Security (DNSSEC)

## Unit 6 - Topic 2 | Network Services Security

---

## Overview

**DNSSEC (DNS Security Extensions)** adds cryptographic authentication to DNS responses, preventing **cache poisoning** and **spoofing** attacks. While standard DNS has no mechanism to verify response authenticity, DNSSEC uses digital signatures to ensure DNS answers come from the legitimate authoritative source and haven't been tampered with.

---

## 1. The Problem DNSSEC Solves

```
┌──────────────────────────────────────────────────────────────┐
│              WITHOUT DNSSEC                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Client asks: "What's the IP of bank.com?"                   │
│                                                              │
│  Legitimate answer: 93.184.216.34                            │
│  Attacker injects:  10.0.0.5 (phishing site!)               │
│                                                              │
│  DNS has NO WAY to verify which answer is real. ❌            │
│                                                              │
├──────────────────────────────────────────────────────────────┤
│              WITH DNSSEC                                      │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  Every DNS response includes a DIGITAL SIGNATURE.            │
│  Resolver verifies signature using public key in DNS.        │
│                                                              │
│  Legitimate answer: 93.184.216.34 + Valid Signature ✅        │
│  Attacker's answer: 10.0.0.5 + Invalid/No Signature ❌       │
│                                                              │
│  Resolver REJECTS unsigned or incorrectly signed responses.  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. How DNSSEC Works

```
DNSSEC Chain of Trust:

Root Zone (.)
  │ Signed with Root KSK
  │ DS record points to .com
  ▼
TLD (.com)
  │ Signed with .com KSK
  │ DS record points to example.com
  ▼
example.com
  │ Signed with example.com ZSK
  │ RRSIG on every record
  ▼
A record: 93.184.216.34 + RRSIG (digital signature)

KEY CONCEPTS:
KSK (Key Signing Key) — Signs the ZSK (rotated infrequently)
ZSK (Zone Signing Key) — Signs the actual DNS records (rotated frequently)
DS (Delegation Signer) — Links parent zone to child zone's key
RRSIG — Digital signature over a DNS record set
NSEC/NSEC3 — Proves a record does NOT exist (authenticated denial)
```

---

## 3. DNSSEC Record Types

| Record | Purpose |
|--------|---------|
| **RRSIG** | Digital signature over DNS records |
| **DNSKEY** | Public key used to verify RRSIG |
| **DS** | Hash of child zone's DNSKEY (delegation signer) |
| **NSEC** | Proves non-existence of a record |
| **NSEC3** | Same as NSEC but hashed (prevents zone walking) |

---

## 4. DNSSEC Limitations

| Limitation | Description |
|-----------|-------------|
| **No encryption** | DNSSEC authenticates but does NOT encrypt DNS queries |
| **Complex deployment** | Key management, signing, rotation is complex |
| **Larger responses** | Signatures increase response size → amplification risk |
| **Zone walking** | NSEC reveals all records (NSEC3 mitigates) |
| **Adoption** | Still not universally deployed |

---

## 5. Beyond DNSSEC — DNS Privacy

| Technology | Purpose | Port |
|------------|---------|:----:|
| **DNSSEC** | Authenticates DNS responses | 53 |
| **DoT (DNS over TLS)** | Encrypts DNS queries | 853 |
| **DoH (DNS over HTTPS)** | Encrypts DNS via HTTPS | 443 |

```
DNSSEC = Integrity + Authentication (is the answer real?)
DoT/DoH = Confidentiality (can anyone see my queries?)

Best: DNSSEC + DoH/DoT = authenticated AND encrypted DNS
```

---

## 6. Verifying DNSSEC

```bash
# Check if domain uses DNSSEC
dig +dnssec example.com
# Look for "ad" flag (Authenticated Data) in response

dig DNSKEY example.com          # View DNSSEC keys
dig DS example.com              # View DS records
dig +trace +dnssec example.com  # Full chain validation

# Online tools
# https://dnsviz.net
# https://dnssec-debugger.verisignlabs.com
```

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **DNSSEC** | Digital signatures on DNS records — prevents poisoning |
| **Provides** | Authentication + Integrity (NOT encryption) |
| **Key Records** | RRSIG, DNSKEY, DS, NSEC/NSEC3 |
| **Limitation** | Complex, no encryption, larger responses |
| **Complement** | DoT/DoH adds encryption on top of DNSSEC |

---

## Quick Revision Questions

1. **What problem does DNSSEC solve?**
2. **What is the difference between KSK and ZSK?**
3. **Does DNSSEC encrypt DNS queries? Why or why not?**
4. **What is the chain of trust in DNSSEC?**
5. **What is the difference between DNSSEC and DoH/DoT?**

---

[← Previous: DHCP Security](01-dhcp-security.md) | [Next: NTP Security →](03-ntp-security.md)

---

*Unit 6 - Topic 2 of 5 | [Back to README](../README.md)*

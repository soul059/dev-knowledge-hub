# Evidence Presentation

## Unit 9 - Topic 7 | Reporting

---

## Overview

**Evidence presentation** is the art of clearly demonstrating that a vulnerability exists and was exploited. Strong evidence makes findings undeniable, helps the client reproduce issues, and protects the pen tester's credibility. Every finding must be supported by verifiable proof.

---

## 1. Types of Evidence

```
┌──────────────────────────────────────────────────────────────┐
│              EVIDENCE TYPES                                   │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SCREENSHOTS                                                 │
│  ├── Most common and impactful                               │
│  ├── Annotated with highlights and arrows                    │
│  └── Include timestamp and context                           │
│                                                              │
│  COMMAND OUTPUT                                               │
│  ├── Exact commands run                                      │
│  ├── Full terminal output                                    │
│  └── Highlight relevant lines                                │
│                                                              │
│  NETWORK CAPTURES                                            │
│  ├── PCAP files showing traffic                              │
│  ├── Relevant packets highlighted                            │
│  └── Used for protocol-level evidence                        │
│                                                              │
│  TOOL REPORTS                                                │
│  ├── Nmap XML, Burp Suite exports                            │
│  ├── Vulnerability scanner reports                           │
│  └── Usually in appendices                                   │
│                                                              │
│  VIDEO                                                       │
│  ├── Complex multi-step exploits                             │
│  ├── Demonstrates full attack chain                          │
│  └── Especially for client presentations                     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. Screenshot Best Practices

```
GOOD SCREENSHOT CHECKLIST:

✅ Shows relevant information clearly
✅ Annotated with arrows/highlights
✅ Numbered to match reproduction steps
✅ Includes timestamp (system clock visible)
✅ Shows target hostname/IP
✅ Sensitive data (PII) is REDACTED
✅ Resolution is readable (not blurry)
✅ Cropped to relevant area
✅ Captioned with description

❌ AVOID:
• Full desktop screenshots (too much noise)
• Tiny text that can't be read
• Missing context (what are we looking at?)
• Unredacted sensitive data
• Screenshots without captions

ANNOTATION EXAMPLE:
┌─────────────────────────────────────────────┐
│ [Screenshot of browser showing admin panel] │
│                                             │
│  ┌──────────────────────────────┐           │
│  │ ← Arrow: "Logged in as      │           │
│  │    admin without valid       │           │
│  │    credentials"              │           │
│  └──────────────────────────────┘           │
│                                             │
│  ┌──────────────────────────────┐           │
│  │ ← Highlight: "47,832        │           │
│  │    customer records          │           │
│  │    accessible"               │           │
│  └──────────────────────────────┘           │
│                                             │
│ Caption: Figure 3 — Database access after   │
│ SQL injection exploitation on login page    │
└─────────────────────────────────────────────┘
```

---

## 3. Command Output Formatting

```
EVIDENCE FORMAT FOR COMMAND OUTPUT:

────────────────────────────────────────────
Command:
$ sqlmap -u "http://10.0.0.5/login?user=admin" --dbs

Output:
[12:34:56] [INFO] testing connection to the target URL
[12:34:57] [INFO] the back-end DBMS is MySQL
[12:34:57] [INFO] fetching database names
available databases [3]:
[*] information_schema
[*] production_db          ← HIGHLIGHTED
[*] mysql

────────────────────────────────────────────
Interpretation:
The SQL injection vulnerability allowed enumeration
of 3 databases. The 'production_db' database contains
the application's customer data.
────────────────────────────────────────────
```

---

## 4. Evidence Integrity

```bash
# MAINTAINING EVIDENCE INTEGRITY:

# 1. Hash all evidence files
sha256sum evidence_screenshot_01.png > evidence_hashes.txt
sha256sum evidence_output_01.txt >> evidence_hashes.txt

# 2. Create evidence manifest
cat evidence_hashes.txt
# a3f2b8c1... evidence_screenshot_01.png
# 7d4e9f3a... evidence_output_01.txt

# 3. Store securely
# Encrypted drive or container
# Access-controlled storage
# No cloud upload without client approval

# 4. Chain of custody
# Document: who collected, when, where stored
# Who had access to evidence

# 5. Retention policy
# Keep for agreed period (usually 30-90 days after report)
# Securely delete after retention period
# Confirm deletion with client
```

---

## 5. Presenting Complex Attack Chains

```
ATTACK CHAIN VISUALIZATION:

STEP 1 ──► STEP 2 ──► STEP 3 ──► STEP 4 ──► STEP 5
RECON       EXPLOIT    PRIVESC    LATERAL    DATA
                                  MOVEMENT   ACCESS

Evidence for each step:

Step 1: [Screenshot: Nmap scan showing open ports]
Step 2: [Output: SQL injection exploitation]
Step 3: [Screenshot: LinPEAS showing SUID binary]
Step 4: [Output: SSH into internal database server]
Step 5: [Screenshot: Customer database query (redacted)]

NARRATIVE:
"Starting from the internet with no credentials,
the tester was able to access the customer database
containing 47,832 records in 5 steps, taking
approximately 2 hours."
```

---

## Summary Table

| Evidence Type | Best For | Format |
|---------------|----------|--------|
| **Screenshot** | Visual proof | PNG, annotated |
| **Command output** | Technical detail | Text, highlighted |
| **PCAP** | Network-level proof | Wireshark export |
| **Video** | Complex attack chains | MP4 for presentations |
| **Tool report** | Supporting data | PDF/XML in appendix |

---

## Quick Revision Questions

1. **Why is evidence important in a pen test report?**
2. **What makes a good screenshot for evidence?**
3. **How should sensitive data be handled in evidence?**
4. **Why hash evidence files?**
5. **How should complex attack chains be presented?**

---

[← Previous: Remediation Recommendations](06-remediation-recommendations.md)

---

*Unit 9 - Topic 7 of 7 | [Back to README](../README.md) | [Next Unit: Remediation Verification →](../10-Remediation-Verification/01-retesting-procedures.md)*

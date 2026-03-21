# MX Record Analysis

## Unit 6 - Topic 3 | Email Reconnaissance

---

## Overview

**MX (Mail Exchange) record analysis** identifies the mail infrastructure of a target organization, revealing mail servers, email providers, security configurations, and potential attack vectors targeting email systems.

---

## 1. MX Record Basics

```bash
# === QUERY MX RECORDS ===
dig target.com MX +short
# 10 mail1.target.com
# 20 mail2.target.com
# 30 mail-backup.target.com

# Priority number = preference (lower = primary)
# 10 = primary mail server
# 20 = secondary (failover)
# 30 = backup

# === DETAILED MX QUERY ===
dig target.com MX +noall +answer
# target.com. 3600 IN MX 10 mail1.target.com.
# target.com. 3600 IN MX 20 mail2.target.com.

# === RESOLVE MX TO IP ===
dig mail1.target.com A +short
# 203.0.113.20

# === REVERSE DNS ===
dig -x 203.0.113.20 +short
# mail1.target.com
```

---

## 2. Email Provider Identification

| MX Record Pattern | Provider |
|------------------|----------|
| `*.google.com` / `*.googlemail.com` | Google Workspace |
| `*.outlook.com` / `*.protection.outlook.com` | Microsoft 365 |
| `*.pphosted.com` | Proofpoint |
| `*.mimecast.com` | Mimecast |
| `*.messagelabs.com` | Broadcom/Symantec |
| `*.barracudanetworks.com` | Barracuda |
| `*.iphmx.com` | Cisco IronPort |
| `*.fireeyecloud.com` | FireEye/Trellix |
| Custom hostname | Self-hosted mail |

```bash
# === EXAMPLE: GOOGLE WORKSPACE ===
dig target.com MX +short
# 1 aspmx.l.google.com
# 5 alt1.aspmx.l.google.com
# → Target uses Google Workspace for email

# === EXAMPLE: MICROSOFT 365 ===
dig target.com MX +short
# 0 target-com.mail.protection.outlook.com
# → Target uses Microsoft 365

# === EXAMPLE: PROOFPOINT (Email Security Gateway) ===
dig target.com MX +short
# 10 target-com.pphosted.com
# → Email flows through Proofpoint first
```

---

## 3. Mail Server Reconnaissance

```bash
# === BANNER GRABBING ===
# Connect to SMTP port to identify software:
telnet mail1.target.com 25
# 220 mail1.target.com ESMTP Postfix (Ubuntu)
# → Server: Postfix on Ubuntu

nc -nv 203.0.113.20 25
# 220 Microsoft ESMTP MAIL Service
# → Microsoft Exchange

# === NMAP SMTP SCRIPTS ===
nmap -p 25,465,587 --script smtp-commands mail1.target.com
# Lists supported SMTP commands

nmap -p 25 --script smtp-ntlm-info mail1.target.com
# If NTLM enabled → reveals internal domain name!

nmap -p 25 --script smtp-open-relay mail1.target.com
# Test for open relay (send mail as anyone)

# === CHECK SUBMISSION PORTS ===
# Port 25:  SMTP (server-to-server)
# Port 465: SMTPS (implicit TLS)
# Port 587: Submission (STARTTLS)
# Port 993: IMAPS
# Port 995: POP3S
nmap -p 25,465,587,993,995 -sV mail1.target.com
```

---

## 4. Intelligence from MX Analysis

```
MX ANALYSIS REPORT:

TARGET: target.com

MX RECORDS:
├── 10 target-com.pphosted.com     → Proofpoint gateway
├── 20 target-com2.pphosted.com    → Proofpoint backup
└── → Actual mail server is BEHIND Proofpoint

EMAIL FLOW:
Internet → Proofpoint (filter) → Exchange (internal) → User

INTELLIGENCE:
├── Proofpoint filters inbound email
│   └── Phishing emails must bypass Proofpoint
├── Exchange server exists internally
│   └── Likely Active Directory environment
├── Two MX records = redundancy
│   └── Backup server may have weaker config
└── Port 587 open = users authenticate to send
    └── Password spray target
```

---

## 5. Attack Vectors from MX Data

| Finding | Attack Vector |
|---------|--------------|
| **Self-hosted mail** | Direct SMTP attacks, exploit server |
| **No email gateway** | Phishing has no filtering |
| **Open relay** | Send spoofed emails through their server |
| **NTLM on SMTP** | Internal domain name disclosure |
| **Port 587 open** | Password spraying for email accounts |
| **Old Exchange** | Known CVEs (ProxyLogon, ProxyShell) |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **MX records** | Identify mail servers and providers |
| **Priority** | Lower number = primary server |
| **Banner grab** | SMTP connection reveals server software |
| **Email gateway** | Proofpoint, Mimecast filter before mail server |
| **Port 587** | Authentication target for password spray |
| **NTLM info** | SMTP NTLM reveals internal domain |

---

## Quick Revision Questions

1. **How do MX record priorities work?**
2. **How do you identify if a target uses Google Workspace vs Microsoft 365?**
3. **What does SMTP banner grabbing reveal?**
4. **Why is an email security gateway important to identify?**
5. **What SMTP port is used for authenticated email submission?**

---

[← Previous: Email Verification](02-email-verification.md) | [Next: SPF, DKIM, and DMARC Analysis →](04-spf-dkim-dmarc-analysis.md)

---

*Unit 6 - Topic 3 of 5 | [Back to README](../README.md)*

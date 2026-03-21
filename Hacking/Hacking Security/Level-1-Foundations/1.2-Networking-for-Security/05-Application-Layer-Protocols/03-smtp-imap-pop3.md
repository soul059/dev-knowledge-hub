# SMTP/IMAP/POP3 (Email Protocols)

## Unit 5 - Topic 3 | Application Layer Protocols

---

## Overview

Email protocols — **SMTP** (sending), **IMAP** and **POP3** (receiving) — are essential communication services that are frequently targeted for **phishing**, **spoofing**, **credential theft**, and **spam relay**. Understanding their security weaknesses and defenses is critical.

---

## 1. Email Protocol Overview

```
┌──────────────────────────────────────────────────────────────────┐
│                  EMAIL FLOW                                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Alice (alice@company.com) sends email to Bob (bob@corp.com)    │
│                                                                  │
│  ┌───────┐  SMTP   ┌──────────┐  SMTP  ┌──────────┐            │
│  │Alice's│────────►│Company's │───────►│ Corp's   │            │
│  │ MUA   │  :587   │Mail Server│  :25  │Mail Server│            │
│  └───────┘         └──────────┘        └────┬─────┘            │
│                                              │                  │
│                                     IMAP :993│or POP3 :995      │
│                                              │                  │
│                                         ┌────┴─────┐            │
│                                         │  Bob's   │            │
│                                         │  MUA     │            │
│                                         └──────────┘            │
│                                                                  │
│  SMTP = Sending/relaying email                                  │
│  IMAP = Reading email (keeps on server)                         │
│  POP3 = Reading email (downloads, deletes from server)          │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 2. Protocol Ports

| Protocol | Cleartext Port | Encrypted Port | Security |
|----------|:-------------:|:--------------:|:--------:|
| **SMTP** | 25 (relay) | 465 (SMTPS) / 587 (STARTTLS) | Encrypt submission |
| **IMAP** | 143 | 993 (IMAPS) | Always use 993 |
| **POP3** | 110 | 995 (POP3S) | Always use 995 |

---

## 3. SMTP Security Issues

| Vulnerability | Description | Defense |
|--------------|-------------|---------|
| **Email Spoofing** | SMTP doesn't verify sender identity | SPF, DKIM, DMARC |
| **Open Relay** | Server relays email from anyone | Restrict relay to authenticated users |
| **Cleartext Auth** | Credentials sent unencrypted on port 25 | STARTTLS or SMTPS |
| **Spam** | Unsolicited bulk email | Anti-spam filters, SPF/DKIM |
| **Phishing** | Fraudulent emails | User training, DMARC |
| **Information Disclosure** | VRFY/EXPN commands reveal users | Disable VRFY and EXPN |

### Email Authentication (SPF, DKIM, DMARC)

```
┌──────────────────────────────────────────────────────────────┐
│              EMAIL AUTHENTICATION STACK                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│  SPF (Sender Policy Framework):                              │
│  DNS TXT record listing authorized sending IPs               │
│  "v=spf1 ip4:10.0.0.1 include:_spf.google.com -all"        │
│  → Prevents sending from unauthorized servers                │
│                                                              │
│  DKIM (DomainKeys Identified Mail):                          │
│  Cryptographic signature in email header                     │
│  Server signs with private key → receiver verifies with DNS  │
│  → Proves email wasn't tampered with in transit              │
│                                                              │
│  DMARC (Domain-based Message Auth):                          │
│  Policy that combines SPF + DKIM                             │
│  "v=DMARC1; p=reject; rua=mailto:reports@company.com"      │
│  p=none    → Monitor only                                   │
│  p=quarantine → Send failures to spam                        │
│  p=reject  → Block failed emails completely                  │
│                                                              │
│  DEFENSE STACK: SPF + DKIM + DMARC = strong email auth      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 4. SMTP Reconnaissance

```bash
# Banner grabbing
nc -v target.com 25
telnet target.com 25
  EHLO test                    # Enumerate capabilities
  VRFY admin                   # Verify if user exists
  EXPN all-users               # Expand mailing list

# Nmap SMTP scripts
nmap -p 25 --script smtp-enum-users target.com
nmap -p 25 --script smtp-open-relay target.com
nmap -p 25 --script smtp-commands target.com

# Check SPF/DKIM/DMARC
dig txt example.com            # SPF record
dig txt _dmarc.example.com    # DMARC policy
dig txt default._domainkey.example.com  # DKIM
```

---

## 5. IMAP vs POP3 Security

| Feature | IMAP | POP3 |
|---------|------|------|
| **Email Storage** | Kept on server | Downloaded to client |
| **Multi-device** | ✅ Syncs across devices | ❌ Single device |
| **Cleartext** | Port 143 (insecure) | Port 110 (insecure) |
| **Encrypted** | Port 993 (IMAPS) | Port 995 (POP3S) |
| **Risk if compromised** | All email accessible on server | Email on client only |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **SMTP** | Email sending — no built-in sender verification |
| **Spoofing Defense** | SPF + DKIM + DMARC |
| **Encrypted Ports** | SMTPS: 465/587, IMAPS: 993, POP3S: 995 |
| **Open Relay** | Major spam risk — always restrict |
| **Recon** | VRFY, EXPN reveal valid users — disable them |

---

## Quick Revision Questions

1. **What are the roles of SMTP, IMAP, and POP3?**
2. **Why is SMTP email spoofing possible?**
3. **Explain SPF, DKIM, and DMARC and how they work together.**
4. **What is an open relay and why is it dangerous?**
5. **Why should VRFY and EXPN commands be disabled?**
6. **What are the encrypted ports for SMTP, IMAP, and POP3?**

---

[← Previous: HTTP/HTTPS](02-http-https.md) | [Next: FTP/SFTP/SCP →](04-ftp-sftp-scp.md)

---

*Unit 5 - Topic 3 of 6 | [Back to README](../README.md)*

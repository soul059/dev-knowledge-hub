# Unit 8: Defense Strategies — Topic 5: Technical Controls

## Overview

**Technical controls** provide automated, technology-based defenses against social engineering attacks. While human awareness is essential, technical controls act as safety nets that catch attacks when humans fail — and as barriers that prevent many attacks from reaching users in the first place. A defense-in-depth approach combines multiple technical controls across email, web, endpoint, network, and identity layers.

---

## 1. Email Security Controls

```
EMAIL DEFENSE LAYERS:

LAYER 1: EMAIL AUTHENTICATION
  SPF (Sender Policy Framework):
    → Specifies authorized sending IPs
    → v=spf1 ip4:x.x.x.x include:_spf.google.com -all
    → Prevents email spoofing
    → "-all" = hard fail (reject unauthorized)

  DKIM (DomainKeys Identified Mail):
    → Cryptographic email signing
    → Verifies email wasn't tampered in transit
    → Public key published in DNS
    → Validates sender domain ownership

  DMARC (Domain-based Message Authentication):
    → Policy enforcement for SPF and DKIM
    → p=none (monitor) → p=quarantine → p=reject
    → Reporting on authentication failures
    → Full deployment = strong spoofing protection

LAYER 2: SECURE EMAIL GATEWAY (SEG)
  → Spam filtering
  → Phishing detection (URL + content analysis)
  → Attachment sandboxing
  → Sender reputation scoring
  → Machine learning-based detection
  → Known threat signature matching
  → BEC detection algorithms

LAYER 3: ADVANCED THREAT PROTECTION
  → URL rewriting and time-of-click scanning
  → Attachment detonation in sandbox
  → Impersonation protection
  → Internal email anomaly detection
  → Post-delivery remediation
  → Automated investigation

LAYER 4: USER-FACING CONTROLS
  → External email banner/warning
  → "This email is from outside your organization"
  → Display name spoofing alerts
  → Report phishing button
  → Suspicious attachment warnings
```

---

## 2. Web and URL Controls

```
WEB SECURITY:

URL FILTERING:
  → Categorize and block malicious sites
  → Real-time URL reputation checking
  → Category blocking (phishing, malware)
  → Newly registered domain blocking
  → Uncategorized domain warnings

SAFE LINKS / URL REWRITING:
  → Rewrite URLs in emails to proxy through security
  → Time-of-click analysis (not just delivery time)
  → Block access if URL becomes malicious after delivery
  → Microsoft Defender Safe Links
  → Proofpoint URL Defense

WEB PROXY:
  → Inspect all web traffic
  → SSL/TLS inspection
  → Content filtering
  → Download scanning
  → Data loss prevention
  → Logging and analytics

DNS SECURITY:
  → Block resolution of known malicious domains
  → DNS sinkholing
  → Category-based filtering
  → Logging DNS queries for analysis
  → Solutions: Cisco Umbrella, Cloudflare Gateway

BROWSER ISOLATION:
  → Execute web content in remote container
  → Only rendered pixels reach user's browser
  → Prevents drive-by downloads
  → Protects against browser exploits
  → Solutions: Menlo, Zscaler, Cloudflare
```

---

## 3. Endpoint Controls

```
ENDPOINT DEFENSE:

ENDPOINT DETECTION AND RESPONSE (EDR):
  → Real-time behavioral monitoring
  → Detect malicious process execution
  → Identify suspicious PowerShell activity
  → Block known malware signatures
  → Automated response and isolation
  → Solutions: CrowdStrike, SentinelOne,
    Microsoft Defender for Endpoint

APPLICATION CONTROL:
  → Whitelist approved applications only
  → Block unsigned/unknown executables
  → Prevent script execution from user folders
  → Windows AppLocker / WDAC
  → Blocks most malware from executing

ATTACK SURFACE REDUCTION:
  → Block Office macro execution
  → Disable PowerShell for standard users
  → Block executable downloads from email
  → Disable auto-run for removable media
  → Prevent process injection
  → Windows ASR rules

USB CONTROLS:
  → Block USB mass storage
  → Whitelist approved devices
  → Block unknown HID devices
  → USB device control policies
  → DLP for removable media
```

---

## 4. Identity and Access Controls

```
IDENTITY PROTECTION:

MULTI-FACTOR AUTHENTICATION (MFA):
  → Phishing-resistant MFA preferred
    - FIDO2/WebAuthn (hardware keys)
    - Certificate-based authentication
  → Standard MFA (still valuable)
    - Authenticator apps (TOTP)
    - Push notifications
  → Avoid: SMS-based MFA (SIM swapping risk)
  → MFA reduces account compromise by 99.9%

CONDITIONAL ACCESS:
  → Location-based access decisions
  → Device compliance requirements
  → Risk-based authentication
  → Session duration limits
  → Step-up authentication for sensitive actions

PASSWORD POLICIES:
  → Minimum length (14+ characters)
  → Password manager encouraged
  → Banned password lists
  → No forced regular rotation
  → Breach detection integration

PRIVILEGED ACCESS MANAGEMENT:
  → Just-in-time access elevation
  → Separate admin accounts
  → Session monitoring and recording
  → Approval workflows for sensitive access
  → Time-limited privileged sessions
```

---

## 5. Network Controls

```
NETWORK DEFENSE:

NETWORK SEGMENTATION:
  → Limit lateral movement after compromise
  → Separate sensitive systems
  → Microsegmentation for critical assets
  → Zero trust network architecture
  → Prevent east-west movement

INTRUSION DETECTION/PREVENTION:
  → Monitor for C2 communication
  → Detect data exfiltration
  → Block known malicious traffic
  → Behavioral anomaly detection
  → Network traffic analysis

DATA LOSS PREVENTION (DLP):
  → Monitor email for sensitive data
  → Block credential sharing via email
  → Prevent unauthorized data transfer
  → Cloud DLP for SaaS applications
  → Endpoint DLP for file transfers

DECEPTION TECHNOLOGY:
  → Honeypots to detect lateral movement
  → Honey tokens (fake credentials)
  → Honey files (fake sensitive documents)
  → Alert on any interaction with decoys
  → Early warning of compromise
```

---

## 6. Defense-in-Depth Summary

```
LAYERED DEFENSE AGAINST SOCIAL ENGINEERING:

LAYER 1: PERIMETER
  Email gateway, web proxy, DNS filtering
  → Blocks 90% of attacks

LAYER 2: ENDPOINT
  EDR, application control, USB policies
  → Catches what passes perimeter

LAYER 3: IDENTITY
  MFA, conditional access, PAM
  → Limits damage from credential theft

LAYER 4: NETWORK
  Segmentation, IDS/IPS, DLP
  → Prevents lateral movement and exfil

LAYER 5: HUMAN
  Training, simulations, reporting
  → The intelligent last line of defense

LAYER 6: MONITORING
  SIEM, SOC, threat intel
  → Detects and responds to what gets through
```

---

## Summary Table

| Control Layer | Technologies | Protects Against |
|--------------|-------------|-----------------|
| Email | SPF/DKIM/DMARC, SEG, ATP | Phishing, spoofing, BEC |
| Web | URL filtering, proxy, DNS | Malicious links, watering holes |
| Endpoint | EDR, app control, ASR | Malware, USB attacks |
| Identity | MFA, conditional access | Credential theft, account takeover |
| Network | Segmentation, DLP, IDS | Lateral movement, data exfiltration |

---

## Revision Questions

1. How do SPF, DKIM, and DMARC work together to prevent email spoofing?
2. What is the difference between a secure email gateway and advanced threat protection?
3. Why is phishing-resistant MFA (FIDO2) preferred over SMS-based MFA?
4. How does browser isolation protect against watering hole attacks?
5. What does a defense-in-depth approach look like for social engineering?

---

*Previous: [04-incident-response.md](04-incident-response.md) | Next: [06-policy-development.md](06-policy-development.md)*

---

*[Back to README](../README.md)*

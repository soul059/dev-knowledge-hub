# Unit 1: Mobile Security Fundamentals — Topic 1: Mobile Threat Landscape

## Overview

The mobile threat landscape has evolved dramatically as smartphones have become the primary computing device for billions of users. Mobile devices store sensitive personal data, financial information, corporate secrets, and authentication credentials — making them **high-value targets** for attackers. Understanding the current threat landscape is essential for mobile security professionals.

---

## 1. Mobile Threat Statistics

```
MOBILE THREAT LANDSCAPE (Current State):

DEVICE PREVALENCE:
├── 6.8+ billion smartphone users worldwide
├── Average user has 80+ apps installed
├── 4+ hours daily mobile usage
├── Mobile banking adoption: 75%+
└── BYOD (Bring Your Own Device): 67% of organizations

THREAT STATISTICS:
├── 500,000+ new mobile malware samples per quarter
├── 1 in 36 mobile devices has high-risk apps installed
├── 46% increase in mobile phishing attacks year-over-year
├── 43% of compromised devices were not jailbroken/rooted
└── Average cost of mobile data breach: $3.5M+
```

---

## 2. Threat Categories

```
MOBILE THREAT TAXONOMY:

┌─────────────────────────────────────────────────┐
│              MOBILE THREATS                      │
├───────────────┬───────────────┬─────────────────┤
│   DEVICE      │   NETWORK     │  APPLICATION    │
│   THREATS     │   THREATS     │  THREATS        │
├───────────────┼───────────────┼─────────────────┤
│ • Jailbreak/  │ • Man-in-    │ • Malicious     │
│   Root exploit│   the-Middle │   apps          │
│ • OS vulns    │ • Rogue WiFi │ • Data leakage  │
│ • Physical    │ • SSL strip  │ • Insecure      │
│   access      │ • DNS spoof  │   storage       │
│ • USB debug   │ • Bluetooth  │ • Weak auth     │
│ • Lost/stolen │   attacks    │ • Code injection│
│ • Outdated OS │ • Cell tower │ • Repackaged    │
│               │   spoofing   │   apps          │
└───────────────┴───────────────┴─────────────────┘

THREAT ACTORS:
├── Script Kiddies → Repackaged malware, phishing
├── Cybercriminals → Banking trojans, ransomware
├── Nation States → Spyware (Pegasus), surveillance
├── Hacktivists → DDoS apps, data leaks
├── Insiders → Data exfiltration via mobile
└── Competitors → Corporate espionage apps
```

---

## 3. Common Attack Vectors

```
ATTACK VECTORS:

1. MALICIOUS APPLICATIONS
   → Repackaged legitimate apps with malware
   → Fake apps mimicking popular brands
   → Trojanized SDKs (supply chain)
   → Side-loaded apps (APK/IPA outside store)

2. PHISHING (SMS/MMS — "Smishing")
   → SMS messages with malicious links
   → Fake login pages optimized for mobile
   → Social engineering via messaging apps
   → QR code phishing ("Quishing")

3. NETWORK-BASED
   → Evil twin WiFi access points
   → SSL/TLS interception on public WiFi
   → BGP hijacking for mobile traffic
   → Cellular network interception (IMSI catchers/Stingrays)

4. PHYSICAL ACCESS
   → Stolen/lost devices
   → USB charging attacks (juice jacking)
   → Shoulder surfing
   → Forensic extraction

5. OS/FIRMWARE EXPLOITS
   → Zero-day iOS/Android exploits
   → Bootloader vulnerabilities
   → Baseband processor exploits
   → Supply chain compromise (factory malware)

6. SOCIAL ENGINEERING
   → Vishing (voice phishing)
   → Impersonation calls
   → Fake tech support
   → SIM swapping attacks
```

---

## 4. Notable Mobile Threats

```
SIGNIFICANT MOBILE THREATS:

PEGASUS (NSO Group):
  → Nation-state level spyware
  → Zero-click exploitation (no user interaction)
  → Full device compromise (calls, messages, location)
  → Targeted journalists, activists, politicians
  → iOS and Android versions

BANKING TROJANS:
  → Cerberus, Anubis, TeaBot, SharkBot
  → Overlay attacks (fake login screens)
  → SMS interception (bypass 2FA)
  → Accessibility service abuse
  → Credential theft

RANSOMWARE:
  → Locks device screen
  → Encrypts files on device
  → Demands cryptocurrency payment
  → Less common than desktop but growing

ADWARE/SPYWARE:
  → Hidden in legitimate-looking apps
  → Collects personal data
  → Tracks location, contacts, messages
  → Often in free apps/games

STALKERWARE:
  → Intimate partner surveillance
  → Hidden monitoring apps
  → Records calls, messages, location
  → Legal/ethical concerns
```

---

## 5. Mobile vs Desktop Threat Differences

| Aspect | Desktop | Mobile |
|--------|---------|--------|
| Attack Surface | Browser, email, files | Apps, SMS, WiFi, physical |
| Malware Distribution | Email, downloads | App stores, side-loading |
| User Interaction | More cautious | Quicker, less careful |
| Permissions | Broad system access | Sandboxed, permission-based |
| Updates | Manual/auto | Carrier/OEM dependency |
| Physical Risk | Lower (stationary) | Higher (portable, lost/stolen) |
| Network | Usually trusted | Often untrusted WiFi |

---

## Revision Questions

1. What are the three main categories of mobile threats?
2. Describe five common mobile attack vectors.
3. What is Pegasus and why is it significant in mobile security?
4. How does the mobile threat landscape differ from desktop?
5. What makes BYOD environments particularly challenging for security?

---

*Previous: None (First topic in this unit) | Next: [02-mobile-vs-web-security.md](02-mobile-vs-web-security.md)*

---

*[Back to README](../README.md)*

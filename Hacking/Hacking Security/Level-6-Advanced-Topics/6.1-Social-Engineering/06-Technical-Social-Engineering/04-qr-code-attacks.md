# Unit 6: Technical Social Engineering — Topic 4: QR Code Attacks

## Overview

**QR code attacks** (also called **quishing** — QR + phishing) exploit the widespread use of QR codes to direct victims to malicious websites, trigger malware downloads, or steal credentials. QR codes are inherently opaque — users cannot read the encoded URL before scanning — making them an effective social engineering vector. The adoption of QR codes accelerated dramatically during COVID-19 for contactless interactions.

---

## 1. How QR Code Attacks Work

```
QR CODE ATTACK FLOW:

ATTACKER:
  ┌─────────────────────────────────┐
  │ 1. Create malicious URL          │
  │ 2. Generate QR code for URL      │
  │ 3. Deliver QR code to victims    │
  │    → Email, poster, sticker,     │
  │      flyer, business card, etc.  │
  └─────────────────────────────────┘
              │
VICTIM:
  ┌─────────────────────────────────┐
  │ 1. Sees QR code                  │
  │ 2. Scans with phone camera       │
  │ 3. Phone opens malicious URL     │
  │ 4. Phishing page / malware       │
  │ 5. Credentials stolen or         │
  │    device compromised            │
  └─────────────────────────────────┘

WHY QR CODES ARE DANGEROUS:
  → Users CANNOT read the URL before scanning
  → Mobile devices have fewer security controls
  → QR codes bypass email URL scanning
  → People trust QR codes (new technology = safe)
  → Redirects happen instantly
  → Phone screen makes URL hard to verify
```

---

## 2. QR Code Attack Vectors

```
ATTACK METHODS:

EMAIL-BASED (Quishing):
  → Email contains QR code image instead of link
  → "Scan to verify your account"
  → "Scan to view the document"
  → Bypasses email URL scanning (it's an image!)
  → Growing rapidly since 2022

PHYSICAL OVERLAY:
  → Place malicious QR sticker over legitimate one
  → Restaurant menus → credential harvesting
  → Parking meters → fake payment page
  → Public transit → malware download
  → Product packaging → fake registration

DOCUMENT-BASED:
  → QR code in PDF or printed document
  → "Scan for additional information"
  → "Scan to access the full report"
  → Trusted document context builds confidence

POSTER/FLYER:
  → Fake advertisements with QR codes
  → "Free WiFi — scan to connect"
  → "Special offer — scan for discount"
  → Placed in high-traffic areas

BUSINESS CARD:
  → QR code on a fake business card
  → Scanned and stored in contacts
  → Links to malicious profile page
  → Dropped or handed out at events

SOCIAL MEDIA:
  → QR codes shared in posts/stories
  → "Scan for exclusive content"
  → Difficult to verify before scanning
  → Viral sharing multiplies reach
```

---

## 3. Technical QR Code Exploits

```
MALICIOUS QR PAYLOADS:

URL REDIRECT:
  → Direct to phishing page
  → Most common attack type
  → Credential harvesting on mobile
  → Mobile-optimized fake login pages

WIFI CONNECTION:
  → QR code auto-connects to rogue WiFi
  → Format: WIFI:T:WPA;S:FreeWiFi;P:password;;
  → Device auto-connects to attacker's AP
  → Man-in-the-middle attacks
  → No user confirmation on some devices

PHONE CALL:
  → QR code initiates phone call
  → Format: tel:+1234567890
  → Calls attacker-controlled number
  → Premium rate numbers (financial loss)
  → Vishing attack setup

SMS MESSAGE:
  → QR code creates pre-filled SMS
  → Format: smsto:+1234567890:Message
  → Sends message to premium number
  → Or sends info to attacker
  → User may not review before sending

CONTACT CARD (vCard):
  → QR code adds contact to phone
  → Includes malicious website in contact
  → Can include fake company info
  → Creates trust for future communication

APP DOWNLOAD:
  → Links to malicious app download
  → Fake app store pages
  → Side-loading malicious APKs
  → "Download our new app!"
```

---

## 4. Real-World QR Code Attacks

```
NOTABLE INCIDENTS:

PARKING METER SCAM (Austin, TX 2022):
  → Malicious QR stickers on parking meters
  → Directed to fake payment page
  → Collected credit card information
  → Spread to multiple cities
  → Difficult to detect (looked official)

QR CODE PHISHING IN EMAILS (2023-2024):
  → 587% increase in QR phishing emails
  → QR code as image bypasses URL scanning
  → Microsoft 365 credential harvesting
  → Targeted corporate employees
  → QR renders on phone, not scanned by email gateway

CRYPTOCURRENCY SCAMS:
  → QR codes at events/meetups
  → "Scan to receive free crypto"
  → Connects to attacker's wallet
  → Victim sends crypto to wrong address
  → Irreversible transactions

COVID-19 EXPLOITS:
  → Fake contact tracing QR codes
  → Fake vaccination verification
  → Restaurant menu QR code overlays
  → "Scan for health information"
  → Exploited pandemic trust in QR codes
```

---

## 5. Defending Against QR Code Attacks

```
INDIVIDUAL DEFENSES:
  → Use QR scanner that previews URL first
  → Check URL domain before proceeding
  → Don't scan QR codes from untrusted sources
  → Look for QR code stickers placed over others
  → Be suspicious of QR codes in emails
  → Use built-in camera app (shows URL preview)
  → Verify QR code context matches expectation

ORGANIZATIONAL DEFENSES:
  → Email security with QR code detection
  → Security awareness training on quishing
  → Restrict QR scanning on corporate devices
  → Monitor for QR code phishing campaigns
  → URL filtering on mobile devices
  → MDM policies for link opening behavior

PHYSICAL DEFENSES:
  → Tamper-evident QR code labels
  → Regular inspection of public QR codes
  → Embed QR codes in design (hard to overlay)
  → Use dynamic QR codes (server-side validation)
  → Report suspicious QR codes to management
```

---

## Summary Table

| Vector | Example | Risk Level | Detection |
|--------|---------|-----------|-----------|
| Email QR (quishing) | "Scan to verify" | High | Hard (image) |
| Physical overlay | Sticker on parking meter | High | Medium |
| Document QR | "Scan for details" | Medium | Hard |
| WiFi QR | Auto-connect to rogue AP | High | Very Hard |
| Fake poster | "Free WiFi/discount" | Medium | Medium |

---

## Revision Questions

1. Why are QR code attacks particularly effective?
2. What types of malicious payloads can QR codes deliver?
3. How does email-based QR phishing bypass security controls?
4. What real-world QR code attacks have occurred?
5. What defenses are most effective against QR code attacks?

---

*Previous: [03-link-manipulation.md](03-link-manipulation.md) | Next: [05-usb-drops.md](05-usb-drops.md)*

---

*[Back to README](../README.md)*

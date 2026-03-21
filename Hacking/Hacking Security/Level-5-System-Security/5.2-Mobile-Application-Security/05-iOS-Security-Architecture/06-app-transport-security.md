# Unit 5: iOS Security Architecture — Topic 6: App Transport Security

## Overview

**App Transport Security (ATS)** is an iOS networking security feature that enforces HTTPS connections with modern TLS configurations. Introduced in iOS 9, ATS blocks insecure HTTP connections by default, requiring developers to explicitly opt out for specific domains. Testing ATS configuration reveals whether apps maintain strong transport security or have weakened it through exceptions.

---

## 1. ATS Architecture

```
ATS ENFORCEMENT:

┌──────────────────────────────────────┐
│           iOS APPLICATION            │
│                                      │
│  NSURLSession / URLSession           │
│  WKWebView                           │
│  Any URL loading system call         │
└──────────────┬───────────────────────┘
               │ network request
┌──────────────▼───────────────────────┐
│      ATS POLICY CHECK                │
│                                      │
│  ✓ HTTPS required (not HTTP)         │
│  ✓ TLS 1.2 or higher                │
│  ✓ Forward secrecy cipher suites     │
│  ✓ Certificate: SHA-256+ signature   │
│  ✓ RSA key: 2048 bits minimum        │
│  ✓ EC key: 256 bits minimum          │
│                                      │
│  PASS → Connection allowed           │
│  FAIL → Connection blocked!          │
└──────────────────────────────────────┘

ATS DEFAULT REQUIREMENTS:
  Protocol:     TLS 1.2+
  Ciphers:      ECDHE + AES-128-GCM or AES-256-GCM
                ECDHE + AES-128-CBC-SHA256
                ECDHE + AES-256-CBC-SHA384
  Key Exchange: ECDHE (forward secrecy required)
  Certificate:  SHA-256 with RSA 2048+ or ECC 256+
```

---

## 2. ATS Configuration in Info.plist

```xml
<!-- SECURE: ATS enabled (default — no config needed) -->
<!-- Just don't add NSAppTransportSecurity at all -->

<!-- INSECURE: ATS completely disabled -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <true/>
    <!-- ALL HTTP connections allowed — FINDING! -->
</dict>

<!-- PARTIAL EXCEPTION: Specific domain -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSExceptionDomains</key>
    <dict>
        <key>legacy-api.example.com</key>
        <dict>
            <key>NSExceptionAllowsInsecureHTTPLoads</key>
            <true/>
            <!-- HTTP allowed for this domain only -->
            <key>NSIncludesSubdomains</key>
            <true/>
        </dict>
    </dict>
</dict>

<!-- MEDIA/WEB: Separate controls -->
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoadsInWebContent</key>
    <true/>
    <!-- WKWebView can load HTTP content -->
    <key>NSAllowsArbitraryLoadsForMedia</key>
    <true/>
    <!-- AVFoundation can load HTTP media -->
</dict>
```

---

## 3. Testing ATS Configuration

```bash
# === STATIC ANALYSIS ===

# Extract Info.plist from IPA
unzip app.ipa -d extracted/
plutil -p extracted/Payload/App.app/Info.plist | grep -A 20 "NSAppTransportSecurity"

# Using objection
objection -g com.example.app explore
> ios plist cat Info.plist

# Using Frida
frida -U -f com.example.app --no-pause -l ats_check.js
```

```
WHAT TO CHECK:

HIGH SEVERITY:
  □ NSAllowsArbitraryLoads = true
    → ATS completely disabled
    → All HTTP traffic allowed

MEDIUM SEVERITY:
  □ NSExceptionAllowsInsecureHTTPLoads for sensitive domains
    → HTTP allowed for specific API servers
  □ NSExceptionMinimumTLSVersion set below TLS 1.2
    → Weak TLS allowed for domain
  □ NSExceptionRequiresForwardSecrecy = false
    → Weakened cipher requirements

LOW SEVERITY:
  □ NSAllowsArbitraryLoadsInWebContent = true
    → Web views can load HTTP (sometimes justified)
  □ NSAllowsArbitraryLoadsForMedia = true
    → Media streaming over HTTP

INFORMATIONAL:
  □ NSAllowsLocalNetworking = true
    → Local network HTTP allowed (development)
```

---

## 4. App Store Review Requirements

```
APPLE'S ATS POLICY:

REQUIRED JUSTIFICATION:
  → Apps disabling ATS must provide justification
  → Apple reviews why ATS exceptions are needed
  → Common accepted reasons:
    • Third-party content (ads, analytics)
    • Legacy server compatibility
    • Media streaming requirements
    • Web browser functionality

REJECTION RISKS:
  → Blanket NSAllowsArbitraryLoads without justification
  → No plan to migrate to HTTPS
  → Sensitive data transmitted over HTTP

BEST PRACTICES:
  1. Keep ATS enabled globally
  2. Use domain-specific exceptions only where needed
  3. Minimize number of exception domains
  4. Plan migration path to remove exceptions
  5. Use NSAllowsArbitraryLoadsInWebContent 
     instead of NSAllowsArbitraryLoads for web views
```

---

## 5. ATS vs. Android Network Security Config

```
COMPARISON:

┌─────────────────┬──────────────────┬──────────────────┐
│ Feature         │ iOS ATS          │ Android NSC      │
├─────────────────┼──────────────────┼──────────────────┤
│ Default         │ HTTPS enforced   │ HTTP allowed*    │
│ Config location │ Info.plist       │ XML file         │
│ Granularity     │ Per-domain       │ Per-domain       │
│ Cert pinning    │ Not in ATS**     │ Supported        │
│ Min TLS         │ 1.2 default      │ Configurable     │
│ Since           │ iOS 9 (2015)     │ Android 7 (2016) │
│ Bypass ease     │ Plist edit       │ XML edit         │
└─────────────────┴──────────────────┴──────────────────┘

*Android 9+ blocks cleartext by default (targetSdkVersion 28+)
**iOS uses separate NSURLSession API for pinning
```

---

## Summary Table

| ATS Setting | Security Impact | Finding Severity |
|------------|----------------|-----------------|
| AllowsArbitraryLoads=true | All HTTP allowed | High |
| Domain HTTP exception | HTTP for specific domain | Medium |
| Min TLS < 1.2 | Weak encryption possible | Medium |
| No forward secrecy | Weaker key exchange | Low-Medium |
| Web content exception | HTTP in web views | Low |
| ATS fully enabled | Strongest transport | N/A (secure) |

---

## Revision Questions

1. What is App Transport Security and what does it enforce?
2. What TLS requirements does ATS mandate by default?
3. How do you check for ATS exceptions in an app's Info.plist?
4. What is the difference between NSAllowsArbitraryLoads and NSAllowsArbitraryLoadsInWebContent?
5. How does ATS compare to Android's Network Security Configuration?

---

*Previous: [05-data-protection-api.md](05-data-protection-api.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

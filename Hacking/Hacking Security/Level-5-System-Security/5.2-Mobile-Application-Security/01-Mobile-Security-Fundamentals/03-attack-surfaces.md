# Unit 1: Mobile Security Fundamentals — Topic 3: Attack Surfaces

## Overview

A mobile application's **attack surface** encompasses all possible entry points an attacker can exploit. Mobile apps have a significantly **larger attack surface** than web applications due to the combination of client-side code, local data storage, hardware access, inter-process communication, and network communication. Mapping the attack surface is the first step in any mobile security assessment.

---

## 1. Mobile Attack Surface Map

```
MOBILE APPLICATION ATTACK SURFACE:

┌──────────────────────────────────────────────────────────┐
│                    MOBILE APP                            │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │ APPLICATION  │  │ DATA STORAGE │  │ NETWORK       │  │
│  │ LAYER        │  │ LAYER        │  │ LAYER         │  │
│  ├──────────────┤  ├──────────────┤  ├───────────────┤  │
│  │• Source code │  │• SQLite DBs  │  │• API calls    │  │
│  │• Binary      │  │• SharedPrefs │  │• WebSockets   │  │
│  │• WebViews    │  │• Keychain    │  │• Push notifs  │  │
│  │• Libraries   │  │• Files/Cache │  │• Cert pinning │  │
│  │• Crypto impl │  │• Clipboard   │  │• DNS          │  │
│  │• Debug code  │  │• Logs        │  │• Proxy detect │  │
│  └──────────────┘  └──────────────┘  └───────────────┘  │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │ PLATFORM     │  │ IPC          │  │ HARDWARE      │  │
│  │ INTERACTION  │  │ LAYER        │  │ LAYER         │  │
│  ├──────────────┤  ├──────────────┤  ├───────────────┤  │
│  │• Permissions │  │• Intents     │  │• GPS/Location │  │
│  │• OS features │  │• URL schemes │  │• Camera/Mic   │  │
│  │• Root/JB det │  │• Content     │  │• Bluetooth    │  │
│  │• Backup      │  │  Providers   │  │• NFC          │  │
│  │• Screenshots │  │• Clipboard   │  │• Biometrics   │  │
│  │• Background  │  │• Deep links  │  │• USB/Debug    │  │
│  └──────────────┘  └──────────────┘  └───────────────┘  │
└──────────────────────────────────────────────────────────┘
```

---

## 2. Attack Surface Details

```
1. APPLICATION LAYER:
   □ Source code (decompiled) — hardcoded secrets
   □ Third-party libraries — known CVEs
   □ WebView components — JavaScript injection
   □ Cryptographic implementation — weak algorithms
   □ Debug/test code left in production
   □ Logging sensitive data (LogCat/Console)
   □ Error handling revealing internals

2. DATA STORAGE:
   □ SQLite databases (unencrypted)
   □ SharedPreferences/NSUserDefaults
   □ Keychain/KeyStore misuse
   □ External storage (world-readable on Android)
   □ App cache and temp files
   □ Clipboard data leakage
   □ Backup files (cloud/local)
   □ Screenshots in app switcher

3. NETWORK COMMUNICATION:
   □ HTTP instead of HTTPS
   □ Weak TLS configuration
   □ Missing certificate pinning
   □ Exposed API endpoints
   □ Insecure WebSocket connections
   □ DNS leakage
   □ Proxy detection bypass

4. INTER-PROCESS COMMUNICATION (IPC):
   □ Exported Activities/Services (Android)
   □ Content Providers without permissions
   □ Broadcast Receivers without protection
   □ URL scheme handling (iOS/Android)
   □ Deep links without validation
   □ Clipboard sharing between apps

5. PLATFORM INTERACTION:
   □ Excessive permissions
   □ Insecure backup configuration
   □ Root/jailbreak detection bypass
   □ Screenshot protection missing
   □ Background task data exposure
   □ Keyboard cache

6. HARDWARE INTERFACES:
   □ GPS tracking without consent
   □ Camera/microphone access
   □ Bluetooth vulnerabilities
   □ NFC data interception
   □ Biometric authentication bypass
   □ USB debugging enabled
```

---

## 3. Attack Surface Assessment Methodology

```
STEP 1: RECONNAISSANCE
  → Identify app functionality
  → Map all features and screens
  → Identify data input points
  → List third-party integrations

STEP 2: STATIC ANALYSIS
  → Decompile application binary
  → Review AndroidManifest.xml / Info.plist
  → Identify exported components
  → Find hardcoded secrets
  → Check library versions for CVEs

STEP 3: DYNAMIC ANALYSIS
  → Monitor network traffic
  → Observe file system changes
  → Track IPC communications
  → Monitor logs and clipboard
  → Test with Frida/Objection

STEP 4: MAP AND PRIORITIZE
  → Create attack surface diagram
  → Prioritize by data sensitivity
  → Focus on authentication/authorization
  → Test high-risk areas first
```

---

## Summary Table

| Surface | Risk Level | Common Issues | Testing Tool |
|---------|-----------|---------------|-------------|
| Source Code | High | Hardcoded secrets, debug code | jadx, Hopper |
| Data Storage | Critical | Unencrypted DB, plaintext creds | File browser, SQLite |
| Network | Critical | No pinning, HTTP, weak TLS | Burp Suite, mitmproxy |
| IPC | High | Exported components, deep links | Drozer, Objection |
| Platform | Medium | Excessive permissions, backups | Manual review |
| Hardware | Medium | Location tracking, biometric bypass | Frida |

---

## Revision Questions

1. Draw a complete mobile application attack surface map.
2. What are the six main attack surface categories for mobile apps?
3. How does the IPC attack surface differ between Android and iOS?
4. What methodology should be followed for attack surface assessment?
5. Which attack surfaces are unique to mobile (not found in web apps)?

---

*Previous: [02-mobile-vs-web-security.md](02-mobile-vs-web-security.md) | Next: [04-owasp-mobile-top-10.md](04-owasp-mobile-top-10.md)*

---

*[Back to README](../README.md)*

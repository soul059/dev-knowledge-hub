# Unit 1: Mobile Security Fundamentals — Topic 2: Mobile vs Web Security

## Overview

While mobile and web applications share some security concerns, mobile apps have **unique characteristics** that create distinct security challenges. Mobile apps operate in sandboxed environments, communicate over untrusted networks, store data locally on devices users physically carry, and interact with hardware sensors — all factors that differentiate mobile security testing from web application testing.

---

## 1. Architecture Comparison

```
WEB APPLICATION:
  ┌──────────┐    HTTPS    ┌──────────┐    SQL    ┌──────────┐
  │ Browser  │────────────▶│  Web     │──────────▶│ Database │
  │ (Client) │◀────────────│  Server  │◀──────────│          │
  └──────────┘             └──────────┘           └──────────┘
  
  → All logic on server side
  → Client is just a browser
  → No local data storage (mostly)
  → Single codebase

MOBILE APPLICATION:
  ┌──────────────┐   HTTPS/WS   ┌──────────┐    SQL    ┌──────────┐
  │ Mobile App   │──────────────▶│  API     │──────────▶│ Database │
  │ ┌──────────┐ │◀──────────────│  Server  │◀──────────│          │
  │ │Local DB  │ │               └──────────┘           └──────────┘
  │ │Keychain  │ │
  │ │Files     │ │   Push
  │ │Prefs     │ │◀─────── Push Server
  │ └──────────┘ │
  │ ┌──────────┐ │
  │ │GPS/Camera│ │
  │ │Contacts  │ │
  │ │Bluetooth │ │
  │ └──────────┘ │
  └──────────────┘
  
  → Logic split: client + server
  → Binary runs on user's device (reversible!)
  → Local data storage (accessible)
  → Platform-specific code (Android/iOS)
  → Hardware access
```

---

## 2. Security Differences

```
KEY DIFFERENCES:

CLIENT-SIDE CODE:
  Web: JavaScript visible in browser, but server logic hidden
  Mobile: Compiled binary on user's device → can be reversed!
  → Secrets in mobile code ARE extractable
  → Never store API keys, passwords, crypto keys in app!

DATA STORAGE:
  Web: Cookies, localStorage (limited)
  Mobile: SQLite databases, SharedPreferences, Keychain,
          files, cache — ALL on user-controlled device
  → All local data accessible on rooted/jailbroken device

NETWORK:
  Web: Usually on trusted networks
  Mobile: WiFi, cellular, constantly switching
  → Must handle untrusted networks
  → Certificate pinning important for mobile

AUTHENTICATION:
  Web: Session cookies, CSRF tokens
  Mobile: OAuth tokens, JWT, biometrics
  → Token storage is critical (Keychain/KeyStore)
  → Session management different (long-lived tokens)

BINARY DISTRIBUTION:
  Web: Server-controlled, always latest version
  Mobile: User-controlled updates, multiple versions active
  → Must support backward compatibility
  → Can't force users to update immediately

PERMISSIONS:
  Web: Limited browser sandbox
  Mobile: Camera, GPS, contacts, SMS, storage access
  → Over-permissioned apps = data leakage risk
```

---

## 3. Testing Approach Differences

| Aspect | Web Testing | Mobile Testing |
|--------|------------|----------------|
| Target | Server-side (mostly) | Client + Server |
| Binary Analysis | N/A | Decompile/reverse APK/IPA |
| Traffic | Proxy browser easily | Proxy + cert pinning bypass |
| Data Storage | Check cookies/localStorage | SQLite, files, Keychain, prefs |
| Authentication | Session cookies | OAuth/JWT tokens, biometrics |
| Code Review | Server source code | Decompiled client code |
| Runtime Analysis | Browser DevTools | Frida, Objection, debugger |
| Encryption | TLS configuration | TLS + local encryption |
| Input Validation | Form inputs, APIs | APIs + IPC (Intents, URL schemes) |
| Environment | Browser only | Emulator + physical device |

---

## 4. Shared Vulnerabilities

```
VULNERABILITIES IN BOTH WEB AND MOBILE:
├── Injection attacks (SQL, command, LDAP)
├── Broken authentication
├── Sensitive data exposure
├── Broken access control
├── Security misconfiguration
├── Insufficient logging
├── Server-side request forgery (SSRF)
└── API vulnerabilities

MOBILE-SPECIFIC VULNERABILITIES:
├── Insecure local data storage
├── Insufficient binary protections
├── Client-side injection (WebView)
├── Reverse engineering of binary
├── Code tampering and repackaging
├── Root/jailbreak detection bypass
├── IPC vulnerabilities (Intents, URL schemes)
├── Clipboard data leakage
├── Screenshot/background data leakage
└── Hardware sensor abuse
```

---

## Summary Table

| Factor | Web App | Mobile App |
|--------|---------|-----------|
| Code Location | Server | Client + Server |
| Code Accessibility | Hidden (server) | Reversible (binary) |
| Data Storage | Server-side | Client-side (device) |
| Network Trust | Usually trusted | Often untrusted |
| Update Control | Instant (server) | User-dependent |
| Attack Surface | Network/browser | Network + device + hardware |
| Testing Tools | Burp, ZAP | Burp + Frida + jadx + MobSF |

---

## Revision Questions

1. How does mobile app architecture differ from web app architecture?
2. Why is client-side code security more critical for mobile apps?
3. What vulnerabilities are unique to mobile applications?
4. How does mobile security testing differ from web security testing?
5. Why is certificate pinning more important for mobile than web?

---

*Previous: [01-mobile-threat-landscape.md](01-mobile-threat-landscape.md) | Next: [03-attack-surfaces.md](03-attack-surfaces.md)*

---

*[Back to README](../README.md)*

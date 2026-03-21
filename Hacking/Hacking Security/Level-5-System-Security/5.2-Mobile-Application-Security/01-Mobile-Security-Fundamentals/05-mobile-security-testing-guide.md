# Unit 1: Mobile Security Fundamentals — Topic 5: Mobile Security Testing Guide (MSTG)

## Overview

The **OWASP Mobile Security Testing Guide (MSTG)** is the comprehensive manual for mobile application security testing. It provides **detailed testing procedures, tools, and techniques** for both Android and iOS platforms. The MSTG is the go-to reference for mobile penetration testers and security auditors, complementing the OWASP Mobile Application Security Verification Standard (MASVS).

---

## 1. MSTG and MASVS Relationship

```
OWASP MOBILE SECURITY FRAMEWORK:

┌────────────────────────────────────────────┐
│         MASVS (What to test)               │
│   Mobile Application Security Verification │
│   Standard — Defines REQUIREMENTS          │
├────────────────────────────────────────────┤
│         MSTG (How to test)                 │
│   Mobile Security Testing Guide —          │
│   Defines TEST PROCEDURES                  │
├────────────────────────────────────────────┤
│     Mobile Top 10 (Awareness)              │
│   Top risks — Defines PRIORITIES           │
└────────────────────────────────────────────┘

MASVS SECURITY LEVELS:
  MASVS-L1: Standard Security
    → Minimum security for all mobile apps
    → Basic security controls
  
  MASVS-L2: Defense-in-Depth
    → Sensitive data handling (banking, healthcare)
    → Additional security layers
  
  MASVS-R: Resilience (Anti-Reverse Engineering)
    → Client-side protection
    → Obfuscation, tamper detection
    → DRM, IP protection
```

---

## 2. MASVS Categories

```
MASVS VERIFICATION CATEGORIES:

MASVS-STORAGE:
  → Secure local data storage
  → Keychain/KeyStore usage
  → Data leakage prevention
  → Logging controls

MASVS-CRYPTO:
  → Proper cryptographic algorithms
  → Key management
  → Random number generation
  → No custom crypto

MASVS-AUTH:
  → Authentication mechanisms
  → Session management
  → Biometric authentication
  → Multi-factor authentication

MASVS-NETWORK:
  → TLS implementation
  → Certificate pinning
  → Secure API communication
  → Network security configuration

MASVS-PLATFORM:
  → Platform-specific security features
  → IPC security
  → WebView configuration
  → Permission model

MASVS-CODE:
  → Code quality
  → Secure compilation
  → Input validation
  → Exception handling

MASVS-RESILIENCE:
  → Anti-tampering
  → Root/jailbreak detection
  → Code obfuscation
  → Emulator detection
```

---

## 3. MSTG Testing Methodology

```
MOBILE SECURITY TESTING WORKFLOW:

PHASE 1: PREPARATION
├── Obtain application (APK/IPA)
├── Set up testing environment
│   ├── Rooted Android device/emulator
│   ├── Jailbroken iOS device
│   ├── Proxy (Burp Suite)
│   └── Testing tools installed
├── Review app documentation
├── Define scope and objectives
└── Set up traffic interception

PHASE 2: INFORMATION GATHERING
├── Map application functionality
├── Identify technology stack
├── Review manifest/Info.plist
├── Identify third-party libraries
├── Map API endpoints
└── Understand data flow

PHASE 3: STATIC ANALYSIS
├── Decompile application
├── Search for hardcoded secrets
├── Review cryptographic usage
├── Analyze permissions
├── Check for debug artifacts
├── Review third-party library versions
└── Identify exported components

PHASE 4: DYNAMIC ANALYSIS
├── Monitor network traffic
├── Test authentication/authorization
├── Check local data storage
├── Test IPC mechanisms
├── Runtime instrumentation (Frida)
├── Test certificate pinning
├── Check logging output
└── Test biometric implementation

PHASE 5: REPORTING
├── Document all findings
├── Rate severity (CVSS)
├── Provide reproduction steps
├── Include screenshots/evidence
├── Recommend remediation
└── Map to OWASP Mobile Top 10
```

---

## 4. Testing Checklist (MSTG-based)

```
ESSENTIAL MSTG TESTS:

DATA STORAGE:
□ Check SharedPreferences/NSUserDefaults for sensitive data
□ Verify SQLite databases are encrypted
□ Check for sensitive data in log files
□ Verify clipboard data handling
□ Check backup configuration
□ Verify screenshot protection
□ Check keyboard cache

CRYPTOGRAPHY:
□ Identify all crypto implementations
□ Verify no weak algorithms (MD5, DES, RC4)
□ Check key storage (Keychain/KeyStore)
□ Verify no hardcoded keys
□ Check random number generation

AUTHENTICATION:
□ Test for client-side auth bypass
□ Verify session timeout
□ Test biometric authentication
□ Check for credential storage
□ Test account lockout

NETWORK:
□ Verify HTTPS everywhere
□ Test certificate pinning
□ Check TLS version/cipher suites
□ Verify no sensitive data in URLs
□ Test proxy detection

PLATFORM:
□ Review exported components (Android)
□ Test URL scheme handling (iOS)
□ Verify WebView configuration
□ Check deep link validation
□ Review permissions

CODE QUALITY:
□ Check for debug code
□ Verify error handling
□ Test input validation
□ Check for stack traces in errors
□ Review third-party libraries
```

---

## 5. MSTG Tool Recommendations

| Category | Android Tools | iOS Tools |
|----------|-------------|-----------|
| Static Analysis | jadx, apktool, MobSF | class-dump, Hopper, MobSF |
| Dynamic Analysis | Frida, Objection, Drozer | Frida, Objection, Cycript |
| Network | Burp Suite, mitmproxy | Burp Suite, Charles Proxy |
| Storage | adb shell, SQLite browser | objection, iExplorer |
| Binary | IDA Pro, Ghidra | IDA Pro, Ghidra, Hopper |
| Automation | MobSF, QARK | MobSF |
| Runtime | Xposed, Frida | Frida, Substrate |

---

## Summary Table

| MASVS Category | Key Tests | Priority |
|---------------|-----------|----------|
| STORAGE | Local data encryption, logging | Critical |
| CRYPTO | Algorithm strength, key management | Critical |
| AUTH | Session management, biometrics | Critical |
| NETWORK | TLS, certificate pinning | High |
| PLATFORM | IPC, WebView, permissions | High |
| CODE | Debug artifacts, input validation | Medium |
| RESILIENCE | Obfuscation, tamper detection | Medium |

---

## Revision Questions

1. What is the relationship between MASVS, MSTG, and Mobile Top 10?
2. What are the three MASVS security levels?
3. Describe the five phases of mobile security testing methodology.
4. List the MASVS verification categories and their key focus areas.
5. What tools are recommended for Android vs. iOS testing?
6. Create a mobile security testing checklist based on MSTG.

---

*Previous: [04-owasp-mobile-top-10.md](04-owasp-mobile-top-10.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

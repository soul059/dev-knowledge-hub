# Unit 1: Mobile Security Fundamentals — Topic 4: OWASP Mobile Top 10

## Overview

The **OWASP Mobile Top 10** is the definitive list of the most critical mobile application security risks. Published by the Open Web Application Security Project (OWASP), it provides a standardized framework for understanding and addressing mobile security vulnerabilities. The 2024 update reflects the evolving mobile threat landscape.

---

## 1. OWASP Mobile Top 10 (2024)

```
OWASP MOBILE TOP 10 — 2024:

┌────┬────────────────────────────────────┐
│ #  │ Vulnerability                      │
├────┼────────────────────────────────────┤
│ M1 │ Improper Credential Usage          │
│ M2 │ Inadequate Supply Chain Security   │
│ M3 │ Insecure Authentication/           │
│    │   Authorization                    │
│ M4 │ Insufficient Input/Output          │
│    │   Validation                       │
│ M5 │ Insecure Communication             │
│ M6 │ Inadequate Privacy Controls        │
│ M7 │ Insufficient Binary Protections    │
│ M8 │ Security Misconfiguration          │
│ M9 │ Insecure Data Storage              │
│ M10│ Insufficient Cryptography          │
└────┴────────────────────────────────────┘
```

---

## 2. Detailed Breakdown

```
M1: IMPROPER CREDENTIAL USAGE
  → Hardcoded credentials in source code
  → API keys embedded in app binary
  → Improper storage of user credentials
  → Shared credentials across environments
  
  EXAMPLE:
  // BAD: Hardcoded API key in source
  private static final String API_KEY = "sk_live_abc123def456";
  
  IMPACT: Complete API access, data breach
  MITIGATION: Server-side key management, environment variables

M2: INADEQUATE SUPPLY CHAIN SECURITY
  → Vulnerable third-party libraries
  → Malicious SDK integration
  → Compromised build pipelines
  → Unsigned or tampered dependencies
  
  EXAMPLE: Malicious analytics SDK exfiltrating user data
  IMPACT: Data theft, malware distribution
  MITIGATION: Dependency scanning, SCA tools, verify signatures

M3: INSECURE AUTHENTICATION/AUTHORIZATION
  → Weak password policies
  → Missing MFA
  → Client-side authentication bypass
  → Broken session management
  → Authorization checks only on client side
  
  EXAMPLE: Modifying user_id in API request to access other accounts
  IMPACT: Account takeover, unauthorized access
  MITIGATION: Server-side validation, proper session management

M4: INSUFFICIENT INPUT/OUTPUT VALIDATION
  → SQL injection via mobile API
  → XSS in WebView components
  → Path traversal
  → Command injection
  
  EXAMPLE: WebView loading untrusted HTML with JS enabled
  IMPACT: Code execution, data theft
  MITIGATION: Input validation, parameterized queries, CSP

M5: INSECURE COMMUNICATION
  → HTTP instead of HTTPS
  → Weak TLS versions (TLS 1.0/1.1)
  → Missing certificate pinning
  → Accepting invalid certificates
  
  EXAMPLE: Sensitive data sent over HTTP on public WiFi
  IMPACT: Man-in-the-Middle, credential theft
  MITIGATION: HTTPS everywhere, certificate pinning, strong TLS

M6: INADEQUATE PRIVACY CONTROLS
  → Excessive data collection
  → Location tracking without consent
  → Sharing data with third-party analytics
  → Insufficient data anonymization
  
  EXAMPLE: App collecting contacts without clear consent
  IMPACT: Privacy violations, regulatory fines (GDPR)
  MITIGATION: Data minimization, clear consent, privacy by design

M7: INSUFFICIENT BINARY PROTECTIONS
  → No code obfuscation
  → No anti-tampering
  → No root/jailbreak detection
  → Debug mode enabled in production
  
  EXAMPLE: Decompiled APK reveals business logic and secrets
  IMPACT: Reverse engineering, cloning, intellectual property theft
  MITIGATION: Obfuscation (ProGuard/R8), tamper detection, RASP

M8: SECURITY MISCONFIGURATION
  → Debug mode in production builds
  → Exported components without protection
  → Insecure backup configuration
  → Default credentials
  → Excessive permissions
  
  EXAMPLE: android:debuggable="true" in production
  IMPACT: Debug access, data extraction
  MITIGATION: Secure build configuration, automated checks

M9: INSECURE DATA STORAGE
  → Unencrypted SQLite databases
  → Sensitive data in SharedPreferences
  → Logging sensitive information
  → Cache containing credentials
  → World-readable files
  
  EXAMPLE: Auth tokens stored in plaintext SharedPreferences
  IMPACT: Credential theft from device
  MITIGATION: Encrypt storage, use Keychain/KeyStore, secure delete

M10: INSUFFICIENT CRYPTOGRAPHY
  → Weak algorithms (MD5, SHA1, DES)
  → Hardcoded encryption keys
  → Improper key management
  → Custom crypto implementations
  → Insufficient key length
  
  EXAMPLE: AES encryption with key hardcoded in app
  IMPACT: Data decryption, bypassed encryption
  MITIGATION: Standard libraries, proper key management, strong algorithms
```

---

## 3. Testing Checklist by Category

| OWASP # | Test | Tool |
|---------|------|------|
| M1 | Search for hardcoded credentials | jadx, grep, MobSF |
| M2 | Check library versions for CVEs | OWASP Dependency-Check |
| M3 | Test auth bypass, IDOR, session mgmt | Burp Suite |
| M4 | Test injection in API and WebView | Burp, Drozer |
| M5 | Check for HTTP, weak TLS, no pinning | Burp, SSLyze |
| M6 | Review permissions, data collection | Manual, MobSF |
| M7 | Check obfuscation, tamper detection | jadx, apktool |
| M8 | Review manifest, backup config | MobSF, manual |
| M9 | Check local storage for sensitive data | File browser, SQLite |
| M10 | Review crypto implementation | Source review, jadx |

---

## Summary Table

| # | Risk | Prevalence | Impact | Prevention |
|---|------|-----------|--------|-----------|
| M1 | Credential Usage | Very Common | Critical | Server-side key mgmt |
| M2 | Supply Chain | Common | Critical | SCA scanning |
| M3 | Auth/Authz | Very Common | Critical | Server-side validation |
| M4 | Input Validation | Common | High | Input sanitization |
| M5 | Communication | Common | High | HTTPS + pinning |
| M6 | Privacy | Common | High | Data minimization |
| M7 | Binary Protection | Very Common | Medium | Obfuscation, RASP |
| M8 | Misconfiguration | Common | High | Secure defaults |
| M9 | Data Storage | Very Common | Critical | Encrypted storage |
| M10 | Cryptography | Common | Critical | Standard crypto libs |

---

## Revision Questions

1. List all 10 items in the OWASP Mobile Top 10 (2024).
2. What is the difference between M3 (Auth) and M1 (Credentials)?
3. How do you test for M9 (Insecure Data Storage) on Android?
4. Why is M7 (Binary Protection) unique to mobile apps?
5. How does M5 (Insecure Communication) differ from web TLS issues?
6. Create a testing checklist covering all 10 OWASP Mobile categories.

---

*Previous: [03-attack-surfaces.md](03-attack-surfaces.md) | Next: [05-mobile-security-testing-guide.md](05-mobile-security-testing-guide.md)*

---

*[Back to README](../README.md)*

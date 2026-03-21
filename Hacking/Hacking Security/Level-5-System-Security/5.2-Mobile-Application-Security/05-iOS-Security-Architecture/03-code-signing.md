# Unit 5: iOS Security Architecture — Topic 3: Code Signing

## Overview

**Code signing** is a fundamental iOS security mechanism that ensures only Apple-approved code runs on iOS devices. Every app, framework, and library must be signed with a valid certificate. This prevents unauthorized code execution, detects tampering, and enforces the App Store distribution model.

---

## 1. Code Signing Chain of Trust

```
CODE SIGNING HIERARCHY:

Apple Root CA
  ↓ signs
Apple Worldwide Developer Relations CA
  ↓ issues
Developer Certificate (per Apple Developer account)
  ↓ signs
Provisioning Profile + App Binary
  ↓ verified by
iOS Kernel (at launch and runtime)

CERTIFICATE TYPES:
┌───────────────────────┬──────────────────────────┐
│ Certificate           │ Purpose                  │
├───────────────────────┼──────────────────────────┤
│ Development           │ Debug on test devices    │
│ Distribution          │ App Store submission     │
│ Ad Hoc                │ Limited device testing   │
│ Enterprise (In-House) │ Internal distribution    │
│ Apple Distribution    │ Unified (Xcode 11+)     │
└───────────────────────┴──────────────────────────┘
```

---

## 2. Provisioning Profiles

```
PROVISIONING PROFILE CONTENTS:

{
  "AppIDName": "com.example.myapp",
  "TeamIdentifier": "ABC123XYZ",
  "CreationDate": "2024-01-01",
  "ExpirationDate": "2025-01-01",
  "Entitlements": {
    "keychain-access-groups": ["ABC123XYZ.*"],
    "get-task-allow": false,
    "aps-environment": "production"
  },
  "DeveloperCertificates": ["<cert_data>"],
  "ProvisionedDevices": ["UDID1", "UDID2"],  // Dev/AdHoc only
  "ProvisionsAllDevices": true                 // Enterprise only
}

PROFILE TYPES:
  Development   → get-task-allow: true (allows debugging)
  Ad Hoc        → Limited device UDIDs
  App Store     → No device restrictions, no debugging
  Enterprise    → ProvisionsAllDevices: true
```

---

## 3. Runtime Code Signing Enforcement

```
WHAT iOS VERIFIES:

AT INSTALL TIME:
  ✓ Valid Apple-rooted certificate chain
  ✓ Provisioning profile matches app
  ✓ Bundle ID matches profile
  ✓ Team ID matches certificate
  ✓ Device UDID in profile (Dev/AdHoc)

AT LAUNCH TIME:
  ✓ Code signature of main binary
  ✓ All embedded frameworks signed
  ✓ Entitlements match profile
  ✓ Code hasn't been modified since signing

AT RUNTIME (CONTINUOUS):
  ✓ Every page of code verified before execution
  ✓ No writable+executable memory (W^X)
  ✓ Dynamic libraries must be signed
  ✓ JIT compilation restricted (except Safari)

AMFI (Apple Mobile File Integrity):
  → Kernel extension that enforces code signing
  → Checks signatures before code execution
  → Verifies entitlements for privileged operations
  → Cannot be disabled without jailbreak
```

---

## 4. Pentesting Implications

```
FOR TESTERS:

SIDELOADING TEST APPS:
  → Need Development certificate + provisioning profile
  → Or Enterprise certificate (risky — can be revoked)
  → Or use Cydia Impactor / AltStore (7-day free cert)
  → Jailbroken device: bypass code signing entirely

RE-SIGNING MODIFIED APPS:
  # Using ldid (jailbroken device)
  ldid -S modified_binary
  
  # Using codesign (macOS)
  codesign -f -s "iPhone Developer: ..." MyApp.app
  
  # Using ios-deploy
  ios-deploy --bundle MyApp.app --debug

CHECKING CODE SIGNING:
  # On macOS
  codesign -dvvv MyApp.app
  # Verify: Authority, TeamID, Entitlements
  
  # Check entitlements
  codesign -d --entitlements :- MyApp.app
  
  # Check provisioning profile
  security cms -D -i embedded.mobileprovision

JAILBREAK IMPACT ON CODE SIGNING:
  → Patches AMFI to allow unsigned code
  → Enables dynamic library injection
  → Allows Frida, Cycript, other tools
  → Required for most iOS security testing
```

---

## 5. Code Signing Weaknesses

```
POTENTIAL ISSUES:

ENTERPRISE CERTIFICATE ABUSE:
  → Enterprise certs meant for internal apps
  → Can distribute to ANY device
  → Malware distributed via enterprise certs
  → Apple can revoke (breaks all apps using it)

EXPIRED CERTIFICATES:
  → Apps with expired certs may not launch
  → Certificate rotation needed
  → Can cause service disruption

ENTITLEMENT MISCONFIGURATION:
  → get-task-allow: true in production
  → Allows debugging production app!
  → Overly broad keychain groups

DEVELOPER PROFILE EXPOSURE:
  → embedded.mobileprovision in IPA
  → Contains Team ID, capabilities, device UDIDs
  → Information disclosure
```

---

## Summary Table

| Mechanism | Purpose | Bypass Requirement |
|-----------|---------|-------------------|
| Code Signing | Integrity verification | Jailbreak / re-sign |
| Provisioning Profile | Authorization | Valid developer account |
| AMFI | Runtime enforcement | Kernel exploit |
| Entitlements | Capability control | Re-sign with new profile |
| App Store Review | Malware prevention | Enterprise cert abuse |

---

## Revision Questions

1. What is the iOS code signing chain of trust?
2. What information is contained in a provisioning profile?
3. How does AMFI enforce code signing at runtime?
4. What are the different certificate types and their purposes?
5. How do penetration testers work around code signing?

---

*Previous: [02-sandbox-model.md](02-sandbox-model.md) | Next: [04-keychain.md](04-keychain.md)*

---

*[Back to README](../README.md)*

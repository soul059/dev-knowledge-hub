# Unit 5: iOS Security Architecture — Topic 2: Sandbox Model

## Overview

The **iOS sandbox** is the primary application isolation mechanism. Every third-party app runs in its own sandbox — a restricted environment that limits file system access, inter-app communication, and system resource usage. Understanding the sandbox model is essential for identifying what data an app can access, how apps communicate, and where sandbox escape vulnerabilities arise.

---

## 1. Sandbox Architecture

```
iOS SANDBOX MODEL:

┌────────────────────────────────────────────────────┐
│                    iOS SYSTEM                       │
│                                                     │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐│
│  │   App A       │  │   App B       │  │  App C    ││
│  │              │  │              │  │           ││
│  │  ┌────────┐  │  │  ┌────────┐  │  │ ┌───────┐││
│  │  │Documents│  │  │  │Documents│  │  │ │Docs   │││
│  │  │Library  │  │  │  │Library  │  │  │ │Library│││
│  │  │tmp      │  │  │  │tmp      │  │  │ │tmp    │││
│  │  └────────┘  │  │  └────────┘  │  │ └───────┘││
│  │              │  │              │  │           ││
│  │  SANDBOX     │  │  SANDBOX     │  │ SANDBOX   ││
│  │  BOUNDARY    │  │  BOUNDARY    │  │ BOUNDARY  ││
│  └──────────────┘  └──────────────┘  └───────────┘│
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │            SHARED SYSTEM RESOURCES               ││
│  │  Contacts, Photos, Camera, Microphone            ││
│  │  (Accessible ONLY with user permission)          ││
│  └─────────────────────────────────────────────────┘│
│                                                     │
│  ┌─────────────────────────────────────────────────┐│
│  │            XNU KERNEL                            ││
│  │  Process isolation, MAC policy enforcement       ││
│  └─────────────────────────────────────────────────┘│
└────────────────────────────────────────────────────┘

SANDBOX RULES:
  ✗ Cannot access other app's sandbox
  ✗ Cannot access arbitrary filesystem locations
  ✗ Cannot directly communicate with other apps
  ✗ Cannot access hardware without permission
  ✓ Full access within own container
  ✓ Controlled access via system APIs
  ✓ Limited IPC via URL schemes, extensions
```

---

## 2. Sandbox Enforcement Mechanisms

```
HOW THE SANDBOX IS ENFORCED:

1. MANDATORY ACCESS CONTROL (MAC):
   → Based on TrustedBSD MAC framework
   → Sandbox profile defines allowed operations
   → Kernel enforces policy — cannot be bypassed from userspace
   → Each app has a profile: container, default, etc.

2. UNIX PERMISSIONS:
   → Each app runs as unique user (mobile)
   → Filesystem permissions restrict access
   → App container owned by app's UID

3. ENTITLEMENTS:
   → Embedded in app's code signature
   → Define capabilities: keychain-access-groups,
     push-notifications, app-groups, etc.
   → Kernel checks entitlements before granting access
   → Cannot be modified without re-signing

4. SANDBOX PROFILES (.sb files):
   → Define allowed syscalls, file paths, network access
   → Written in Scheme-like language (SBPL)
   → System apps have custom profiles
   → Third-party apps use "container" profile
   
   ; Example sandbox profile rule
   (allow file-read*
       (subpath "/usr/lib")
       (subpath "/System/Library"))
   (deny file-write*
       (subpath "/System"))
```

---

## 3. Inter-App Communication (Controlled)

```
ALLOWED IPC MECHANISMS:

URL SCHEMES:
  → App registers custom URL scheme
  → Other apps can open URLs: myapp://action?param=value
  → One-way communication (no return data in classic scheme)
  → Universal Links (HTTPS-based, more secure)

APP GROUPS:
  → Apps from same developer share a container
  → Entitlement: com.apple.security.application-groups
  → Shared UserDefaults, files, Core Data
  → Used for app + extension communication

EXTENSIONS:
  → Share Extension, Today Widget, etc.
  → Run in separate process with own sandbox
  → Limited communication with host app
  → Restricted API access

PASTEBOARD (CLIPBOARD):
  → Shared across apps (privacy concern)
  → iOS 14+: notification when app reads clipboard
  → Named pasteboards for app group sharing

KEYCHAIN SHARING:
  → Apps with same keychain-access-group entitlement
  → Share credentials via Keychain
  → Requires same Team ID or explicit group
```

---

## 4. Sandbox Escape Vulnerabilities

```
SANDBOX ESCAPE TYPES:

KERNEL EXPLOITS:
  → Vulnerability in XNU kernel
  → Escape sandbox via kernel code execution
  → Required for jailbreaking
  → Examples: IOKit driver bugs, use-after-free

IPC VULNERABILITIES:
  → Mach port manipulation
  → XPC service vulnerabilities
  → Exploit system daemon via IPC
  → Gain elevated privileges

FILE SYSTEM RACE CONDITIONS:
  → TOCTOU (Time of Check/Time of Use)
  → Symlink attacks on sandbox boundaries
  → Exploit file operations during access checks

LOGIC BUGS:
  → Bypassing sandbox checks via logic errors
  → Misconfigurations in sandbox profiles
  → Incorrect entitlement checking

IMPACT OF SANDBOX ESCAPE:
  → Access other apps' data
  → Read keychain entries
  → Install persistent backdoors
  → Full device compromise
  → Required step in jailbreak chains
```

---

## Summary Table

| Enforcement Layer | Mechanism | Bypass Difficulty |
|------------------|-----------|-------------------|
| MAC Policy | TrustedBSD sandbox profiles | Very Hard (kernel) |
| UNIX Permissions | UID/GID file ownership | Hard (need root) |
| Entitlements | Code signature embedded | Hard (need re-sign) |
| ASLR | Address randomization | Medium (info leak needed) |
| Code Signing | Signature verification | Hard (Apple cert needed) |

---

## Revision Questions

1. What is the iOS sandbox and how does it isolate apps?
2. How are sandbox policies enforced at the kernel level?
3. What IPC mechanisms are available for inter-app communication?
4. What are entitlements and how do they extend sandbox capabilities?
5. What types of vulnerabilities enable sandbox escape?

---

*Previous: [01-ios-architecture.md](01-ios-architecture.md) | Next: [03-code-signing.md](03-code-signing.md)*

---

*[Back to README](../README.md)*

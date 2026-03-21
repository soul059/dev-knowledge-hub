# Unit 5: iOS Security Architecture — Topic 1: iOS Architecture

## Overview

Understanding the **iOS architecture** is essential for mobile security testing. iOS is built on a layered design with security embedded at every level — from hardware (Secure Enclave) to software (sandboxing, code signing). This topic covers the complete iOS system architecture, security features, and how they affect penetration testing.

---

## 1. iOS System Architecture Layers

```
iOS ARCHITECTURE STACK:

┌─────────────────────────────────────────────┐
│         APPLICATION LAYER                    │
│   ┌─────────┐ ┌─────────┐ ┌─────────┐      │
│   │App Store│ │Custom   │ │System   │      │
│   │  Apps   │ │  Apps   │ │  Apps   │      │
│   └────┬────┘ └────┬────┘ └────┬────┘      │
├────────┴──────────┴──────────┴──────────────┤
│         COCOA TOUCH FRAMEWORK               │
│   UIKit, WebKit, MapKit, HealthKit,         │
│   Notifications, Multitasking               │
├─────────────────────────────────────────────┤
│         MEDIA LAYER                          │
│   Core Graphics, Core Audio, AVFoundation,  │
│   Metal, Core Image, Core Animation         │
├─────────────────────────────────────────────┤
│         CORE SERVICES                        │
│   Foundation, Core Data, Core Location,     │
│   CloudKit, Core Motion, Security Framework │
├─────────────────────────────────────────────┤
│         CORE OS / DARWIN                     │
│   XNU Kernel (Mach + BSD), POSIX,           │
│   Libsystem, Security, Keychain,            │
│   Accelerate, IOKit                          │
├─────────────────────────────────────────────┤
│         HARDWARE & FIRMWARE                  │
│   Secure Enclave, Secure Boot Chain,        │
│   Touch ID / Face ID, NFC, Baseband,        │
│   AES Engine, UID/GID Keys                  │
└─────────────────────────────────────────────┘
```

---

## 2. Key Security Components

```
HARDWARE SECURITY:

SECURE ENCLAVE PROCESSOR (SEP):
  → Separate coprocessor with own OS (sepOS)
  → Handles biometric data, encryption keys
  → Generates and stores device unique ID (UID)
  → Key operations never leave SEP
  → Cannot be read by main processor
  → Used for: Touch ID, Face ID, Apple Pay

SECURE BOOT CHAIN:
  Boot ROM (hardware root of trust)
     ↓ verifies
  iBoot (bootloader)
     ↓ verifies
  XNU Kernel
     ↓ verifies
  System libraries and apps
  
  → Each stage verifies the next
  → If verification fails → device won't boot
  → Prevents running modified OS

AES HARDWARE ENGINE:
  → Dedicated hardware for AES encryption
  → File-level encryption (Data Protection)
  → Uses per-file keys wrapped with class keys
  → Class keys wrapped with device UID key

SOFTWARE SECURITY:

XNU KERNEL:
  → Hybrid kernel (Mach microkernel + BSD)
  → Kernel Integrity Protection (KIP)
  → Pointer Authentication Codes (PAC)
  → Page Protection Layer (PPL)
  → Kernel Address Space Layout Randomization

APP SECURITY:
  → Mandatory code signing
  → App sandbox (no access to other apps)
  → Entitlements-based capability system
  → App Store review process
  → App Transport Security (ATS)
```

---

## 3. iOS File System Structure

```
iOS FILE SYSTEM (Jailbroken):

/                              Root filesystem
├── Applications/              System apps (Safari, Mail)
├── Developer/                 Development tools
├── Library/
│   ├── Preferences/           System-wide preferences
│   └── Keychains/             Keychain database
├── System/
│   └── Library/
│       ├── Frameworks/        System frameworks
│       ├── PrivateFrameworks/  Private APIs
│       └── CoreServices/      Core system services
├── private/
│   └── var/
│       ├── mobile/            User data partition
│       │   ├── Applications/  Sideloaded apps
│       │   ├── Containers/
│       │   │   ├── Bundle/    App bundles (.app)
│       │   │   ├── Data/      App data (sandbox)
│       │   │   └── Shared/    App group containers
│       │   ├── Library/
│       │   │   ├── Preferences/  User preferences
│       │   │   └── SMS/          Messages database
│       │   └── Media/         Photos, videos
│       └── db/                System databases
└── usr/
    ├── bin/                   System binaries
    ├── lib/                   System libraries
    └── sbin/                  System admin binaries

APP SANDBOX STRUCTURE:
/var/mobile/Containers/Data/Application/<UUID>/
├── Documents/                 User-visible documents
├── Library/
│   ├── Caches/                Cached data
│   ├── Preferences/           NSUserDefaults (.plist)
│   ├── Application Support/   App support files
│   └── Cookies/               Cookie storage
├── SystemData/                System-managed data
├── tmp/                       Temporary files
└── StoreKit/                  In-app purchase receipts
```

---

## 4. Security Features vs. Android

```
COMPARISON:

┌──────────────────┬──────────────────┬──────────────────┐
│ Feature          │ iOS              │ Android          │
├──────────────────┼──────────────────┼──────────────────┤
│ App Distribution │ App Store only   │ Multiple sources │
│ Code Signing     │ Mandatory (Apple)│ Self-signed OK   │
│ Sandboxing       │ Strict sandbox   │ Linux permissions│
│ Root Access      │ Jailbreak needed │ Rooting (varies) │
│ File Encryption  │ Hardware AES     │ Software/HW AES  │
│ Kernel           │ XNU (Mach+BSD)   │ Linux kernel     │
│ Biometric Store  │ Secure Enclave   │ TEE / StrongBox  │
│ IPC              │ URL Schemes, XPC │ Intents, Binder  │
│ Permissions      │ Per-use prompts  │ Install/runtime  │
│ Updates          │ Centralized      │ Fragmented       │
│ Source Code       │ Closed source    │ AOSP (open core) │
│ Debug            │ Harder (no adb)  │ Easier (adb)     │
└──────────────────┴──────────────────┴──────────────────┘
```

---

## Summary Table

| Component | Security Role | Pentesting Impact |
|-----------|--------------|-------------------|
| Secure Enclave | Key storage, biometrics | Cannot extract keys |
| Secure Boot | Integrity chain | Prevents persistent rootkits |
| Code Signing | App authenticity | Must re-sign modified apps |
| Sandbox | App isolation | Limits lateral movement |
| AES Engine | File encryption | Data protected at rest |
| ASLR/PAC | Memory protection | Harder exploitation |

---

## Revision Questions

1. What are the main layers of the iOS architecture?
2. How does the Secure Enclave protect sensitive data?
3. What is the iOS secure boot chain?
4. How does iOS app sandboxing differ from Android?
5. What is the app sandbox directory structure?

---

*Previous: None (First topic in this unit) | Next: [02-sandbox-model.md](02-sandbox-model.md)*

---

*[Back to README](../README.md)*

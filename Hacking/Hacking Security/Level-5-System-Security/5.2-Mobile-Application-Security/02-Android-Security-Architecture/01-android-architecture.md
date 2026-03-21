# Unit 2: Android Security Architecture — Topic 1: Android Architecture

## Overview

Understanding the **Android system architecture** is fundamental for mobile security testing. Android is built on a **modified Linux kernel** with multiple layers that each provide security capabilities. Knowing how these layers interact helps security professionals identify where vulnerabilities occur and how to exploit or defend against them.

---

## 1. Android Architecture Layers

```
ANDROID ARCHITECTURE STACK:

┌─────────────────────────────────────────┐
│          APPLICATIONS                   │
│  (System apps, User apps, Third-party)  │
├─────────────────────────────────────────┤
│       JAVA/KOTLIN API FRAMEWORK         │
│  (Activity Manager, Content Providers,  │
│   Package Manager, Window Manager,      │
│   Telephony Manager, Resource Manager)  │
├────────────────────┬────────────────────┤
│  NATIVE C/C++      │  ANDROID RUNTIME   │
│  LIBRARIES          │  (ART)             │
│  ├─ OpenSSL/BoringSSL│  ├─ DEX bytecode  │
│  ├─ SQLite          │  ├─ AOT compile   │
│  ├─ WebKit/Chromium │  ├─ Garbage       │
│  ├─ Media Framework │  │  collection    │
│  ├─ libc (Bionic)   │  └─ Core libs     │
│  └─ libcrypto       │                    │
├────────────────────┴────────────────────┤
│     HARDWARE ABSTRACTION LAYER (HAL)    │
│  (Camera, Bluetooth, Sensors, Audio)    │
├─────────────────────────────────────────┤
│          LINUX KERNEL                   │
│  (Process mgmt, Memory mgmt, Security, │
│   Networking, Drivers, Binder IPC)      │
└─────────────────────────────────────────┘
```

---

## 2. Key Components for Security

```
LINUX KERNEL SECURITY:
├── Process isolation (each app = separate Linux user)
├── File system permissions (UID/GID based)
├── SELinux (Mandatory Access Control)
├── seccomp (system call filtering)
├── Network namespace isolation
└── ASLR, stack canaries, NX bit

ANDROID RUNTIME (ART):
├── Replaces Dalvik VM (Android 5.0+)
├── Ahead-of-Time (AOT) compilation
├── DEX bytecode → native machine code
├── Better performance and security
├── Garbage collection improvements
└── Improved debugging and profiling

BINDER IPC:
├── Inter-Process Communication mechanism
├── All communication between apps goes through Binder
├── Kernel-level mediation of all IPC
├── Permission checks at Binder level
├── Used for: Activities, Services, Content Providers
└── Security boundary between processes

APPLICATION FRAMEWORK:
├── Activity Manager — Manages app lifecycle
├── Package Manager — Installs and tracks apps
├── Content Providers — Structured data sharing
├── Window Manager — Manages app windows
├── Telephony Manager — Phone/SMS access
└── Location Manager — GPS access
```

---

## 3. Android File System

```
KEY DIRECTORIES:

/data/data/<package_name>/      ← App's private data directory
├── shared_prefs/               ← SharedPreferences XML files
├── databases/                  ← SQLite databases
├── files/                      ← Internal files
├── cache/                      ← Cache files
└── lib/                        ← Native libraries

/data/app/<package_name>/       ← Installed APK files
/data/system/                   ← System databases, config
/sdcard/ (or /storage/emulated/0/)  ← External storage (shared!)
/system/                        ← System files (read-only)
/system/app/                    ← Pre-installed system apps
/system/framework/              ← Core framework JARs

SECURITY NOTE:
  /data/data/<pkg>/ → Only accessible by the app (UID-based)
  /sdcard/ → Readable by ALL apps with storage permission!
  → NEVER store sensitive data on external storage!
```

---

## 4. Android Versions and Security Features

| Android Version | API Level | Key Security Features |
|----------------|-----------|----------------------|
| 5.0 Lollipop | 21 | ART runtime, SELinux enforcing |
| 6.0 Marshmallow | 23 | Runtime permissions, fingerprint API |
| 7.0 Nougat | 24 | Network Security Config, file-based encryption |
| 8.0 Oreo | 26 | Background limits, autofill framework |
| 9.0 Pie | 28 | Biometric API, TLS by default |
| 10 | 29 | Scoped storage, TLS 1.3 |
| 11 | 30 | One-time permissions, scoped storage enforced |
| 12 | 31 | Approximate location, app hibernation |
| 13 | 33 | Photo picker, notification permissions |
| 14 | 34 | Credential Manager, partial media access |

---

## Summary Table

| Layer | Security Role | Attack Vector |
|-------|-------------|---------------|
| Linux Kernel | Process isolation, SELinux | Kernel exploits, root |
| HAL | Hardware access control | Driver vulnerabilities |
| Native Libraries | Crypto, media processing | Buffer overflows, CVEs |
| ART | Code execution, sandboxing | DEX manipulation |
| Framework | Permission enforcement | Framework bypass |
| Applications | User functionality | App-level vulnerabilities |

---

## Revision Questions

1. Draw and explain the Android architecture stack.
2. How does the Linux kernel provide security for Android apps?
3. What is the role of Binder IPC in Android security?
4. Why is ART more secure than the old Dalvik VM?
5. What are the key directories in the Android file system for security testing?

---

*Previous: None (First topic in this unit) | Next: [02-application-sandbox.md](02-application-sandbox.md)*

---

*[Back to README](../README.md)*

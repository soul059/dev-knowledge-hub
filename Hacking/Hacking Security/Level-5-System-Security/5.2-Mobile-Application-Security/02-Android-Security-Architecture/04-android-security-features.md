# Unit 2: Android Security Architecture — Topic 4: Android Security Features

## Overview

Android includes numerous **built-in security features** at multiple layers of the stack. These features work together to protect device integrity, user data, and application isolation. Understanding these features helps security testers identify what protections exist and how they might be circumvented.

---

## 1. Boot Security

```
VERIFIED BOOT (Android Verified Boot — AVB):

BOOT CHAIN:
  ROM Bootloader → Verified Bootloader → Verified Kernel → 
  Verified System → Verified Vendor

  → Each stage verifies the next before loading
  → Cryptographic verification using RSA/SHA256
  → Detects tampering of system partition
  → dm-verity: Verifies system partition at block level
  → Rollback protection prevents downgrade attacks

  STATES:
  GREEN:  Fully verified, OEM key
  YELLOW: Verified with custom key (custom ROM)
  ORANGE: Bootloader unlocked (development)
  RED:    Verification failed (tampered)
```

---

## 2. Data Protection

```
FILE-BASED ENCRYPTION (FBE):
  → Each file encrypted with unique key
  → Different encryption for:
    CE (Credential Encrypted): After user unlocks
    DE (Device Encrypted): Available at boot
  → AES-256-XTS for file contents
  → AES-256-CTS for file names
  → Hardware-backed key storage

FULL DISK ENCRYPTION (FDE) — Legacy:
  → Entire /data partition encrypted
  → Single key derived from user PIN/password
  → Replaced by FBE in Android 7.0+

HARDWARE-BACKED KEYSTORE:
  → Keys stored in TEE (Trusted Execution Environment)
  → Or Strongbox (separate hardware security module)
  → Keys never leave secure hardware
  → Resistant to OS-level compromise
  → Biometric binding available

ANDROID KEYSTORE SYSTEM:
  → API for apps to store cryptographic keys
  → Keys bound to device, user authentication
  → Hardware-backed when available
  → Supports: AES, RSA, EC, HMAC
  → Key attestation (prove key is hardware-backed)
```

---

## 3. Runtime Security

```
ADDRESS SPACE LAYOUT RANDOMIZATION (ASLR):
  → Randomizes memory addresses
  → Stack, heap, libraries at random locations
  → Makes exploitation harder
  → Full ASLR since Android 4.1

STACK CANARIES:
  → Detect stack buffer overflows
  → Random value placed before return address
  → Checked before function return

NX BIT (No-Execute):
  → Memory pages marked non-executable
  → Prevents code execution from stack/heap
  → Hardware-enforced (ARM XN bit)

CONTROL FLOW INTEGRITY (CFI):
  → Ensures function calls go to valid targets
  → Prevents ROP/JOP attacks
  → Applied to system libraries (Android 8.1+)

MEMORY TAGGING EXTENSION (MTE) — Android 14+:
  → Hardware-based memory safety
  → Detects use-after-free, buffer overflow
  → ARM MTE support
```

---

## 4. Network Security

```
NETWORK SECURITY CONFIGURATION (Android 7.0+):
  → XML-based network security settings per app
  → Controls which CAs are trusted
  → Certificate pinning configuration
  → Cleartext traffic restrictions

  <!-- res/xml/network_security_config.xml -->
  <network-security-config>
    <domain-config cleartextTrafficPermitted="false">
      <domain includeSubdomains="true">api.example.com</domain>
      <pin-set>
        <pin digest="SHA-256">base64_hash_here</pin>
        <pin digest="SHA-256">backup_pin_here</pin>
      </pin-set>
    </domain-config>
    <base-config cleartextTrafficPermitted="false">
      <trust-anchors>
        <certificates src="system" />
      </trust-anchors>
    </base-config>
  </network-security-config>

DEFAULT BEHAVIOR BY TARGET SDK:
  SDK 28+ (Android 9): Cleartext HTTP blocked by default
  SDK 24+ (Android 7): User-added CAs not trusted by default
```

---

## 5. Application Security Features

```
GOOGLE PLAY PROTECT:
  → Scans apps before and after installation
  → Machine learning-based malware detection
  → Verifies apps on Play Store and side-loaded
  → Can remotely disable malicious apps

SAFETYNET / PLAY INTEGRITY API:
  → Verifies device integrity
  → Detects: root, custom ROM, emulator, bootloader unlock
  → Apps can check if device is trustworthy
  → Used by banking and DRM apps

  Checks:
  ├── CTS profile match (device passes Compatibility Test Suite)
  ├── Basic integrity (device not tampered)
  ├── Hardware-backed attestation (strong verification)
  └── App licensing verification

SCOPED STORAGE (Android 10+):
  → Apps can only access their own files by default
  → MediaStore API for shared media access
  → No direct access to other apps' files
  → Limits data harvesting from device
```

---

## Summary Table

| Feature | Since | Protection | Bypass Difficulty |
|---------|-------|-----------|-------------------|
| Verified Boot | 4.4 | Boot integrity | Hard (hardware) |
| FBE | 7.0 | Data at rest | Hard (crypto) |
| SELinux | 5.0 | Access control | Medium-Hard |
| ASLR | 4.1 | Memory safety | Medium |
| Network Security Config | 7.0 | TLS enforcement | Medium (if misconfigured) |
| Play Integrity | Ongoing | Device integrity | Medium |
| Scoped Storage | 10 | File isolation | Low-Medium |
| Hardware KeyStore | 6.0 | Key protection | Very Hard |

---

## Revision Questions

1. How does Android Verified Boot protect the boot chain?
2. What is the difference between FBE and FDE?
3. How does the Android KeyStore system protect cryptographic keys?
4. What does the Network Security Configuration control?
5. How does Play Integrity API verify device trustworthiness?

---

*Previous: [03-permissions-model.md](03-permissions-model.md) | Next: [05-apk-structure.md](05-apk-structure.md)*

---

*[Back to README](../README.md)*

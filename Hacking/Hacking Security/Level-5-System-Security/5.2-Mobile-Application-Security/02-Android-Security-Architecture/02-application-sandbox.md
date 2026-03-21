# Unit 2: Android Security Architecture — Topic 2: Application Sandbox

## Overview

The Android **application sandbox** is the foundational security mechanism that **isolates each application** from other apps and the system. Based on Linux user isolation, each app runs as a separate Linux user with its own UID, ensuring that one app cannot access another app's data or code without explicit permission.

---

## 1. How the Sandbox Works

```
ANDROID SANDBOX MODEL:

┌─────────────────────────────────────────────────┐
│                  ANDROID OS                      │
│                                                  │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │  App A   │  │  App B   │  │  App C   │      │
│  │ UID:10045│  │ UID:10046│  │ UID:10047│      │
│  │          │  │          │  │          │      │
│  │ /data/   │  │ /data/   │  │ /data/   │      │
│  │ data/    │  │ data/    │  │ data/    │      │
│  │ pkg.a/   │  │ pkg.b/   │  │ pkg.c/   │      │
│  │ ═════    │  │ ═════    │  │ ═════    │      │
│  │ ISOLATED │  │ ISOLATED │  │ ISOLATED │      │
│  └──────────┘  └──────────┘  └──────────┘      │
│       │              │              │            │
│       └──────────────┼──────────────┘            │
│                      │                           │
│              LINUX KERNEL                        │
│        (UID-based file permissions)              │
│        (SELinux MAC enforcement)                 │
│        (Seccomp syscall filtering)               │
└─────────────────────────────────────────────────┘

KEY PRINCIPLES:
1. Each app = unique Linux UID
2. Each app's data = owned by that UID
3. File permissions: owner read/write only (0700)
4. No app can read another app's files (by default)
5. IPC goes through Binder (kernel-mediated)
```

---

## 2. Sandbox Components

```
LINUX USER ISOLATION:
  → App A installed → assigned UID 10045
  → /data/data/com.example.appa → owned by UID 10045
  → Only UID 10045 processes can access these files
  → Other apps get "Permission denied"

PROCESS ISOLATION:
  → Each app runs in its own process
  → Separate virtual memory space
  → Cannot access other processes' memory
  → Crashes in one app don't affect others

SELinux (Security-Enhanced Linux):
  → Mandatory Access Control (MAC)
  → Even root cannot bypass SELinux policies
  → Each app has SELinux context/label
  → Policies define exactly what each context can do
  → Enforcing mode since Android 5.0

SECCOMP (Secure Computing):
  → Restricts which system calls apps can make
  → Reduces kernel attack surface
  → Blocks dangerous syscalls (e.g., reboot, mount)
  → Applied to all apps since Android 8.0

SHARED UID (Exception):
  → Apps signed with same key CAN share UID
  → android:sharedUserId in manifest
  → Allows data sharing between related apps
  → Deprecated in Android 10+
```

---

## 3. Breaking the Sandbox

```
HOW ATTACKERS ESCAPE THE SANDBOX:

1. ROOT ACCESS:
   → Root (UID 0) bypasses file permissions
   → Can read ANY app's data
   → SELinux may still block some access
   → Rooted device = sandbox broken

2. KERNEL EXPLOITS:
   → Exploit kernel vulnerability
   → Gain arbitrary code execution in kernel
   → Bypass all sandbox protections
   → Examples: Dirty COW (CVE-2016-5195)

3. EXPORTED COMPONENTS:
   → App explicitly exports Activities/Services
   → Other apps can interact via Intents
   → Misconfigured exports = data leakage
   → Not a sandbox break, but authorized cross-app access

4. CONTENT PROVIDERS:
   → Shared data mechanism between apps
   → Misconfigured permissions → unauthorized access
   → SQL injection in content providers

5. EXTERNAL STORAGE:
   → /sdcard/ is shared between all apps
   → Any app with READ_EXTERNAL_STORAGE can access
   → Sensitive data on SD card = sandbox bypass

6. BACKUP EXTRACTION:
   → adb backup extracts app data
   → android:allowBackup="true" (default!)
   → Attacker with USB access can extract data
```

---

## 4. Security Testing the Sandbox

```bash
# Check app UID
adb shell ps | grep com.example.app
# Output: u0_a45  12345 ...
# u0_a45 = UID 10045

# Check file permissions
adb shell ls -la /data/data/com.example.app/
# drwx------ u0_a45 u0_a45 ... shared_prefs
# drwx------ u0_a45 u0_a45 ... databases

# Check if files are world-readable (vulnerability!)
adb shell find /data/data/com.example.app/ -perm -o+r
# Should return nothing — world-readable = vulnerability

# Check SELinux status
adb shell getenforce
# Should return: Enforcing

# Check app's SELinux context
adb shell ps -Z | grep com.example.app
# u:r:untrusted_app:s0:c45 ...

# Check backup configuration
# In AndroidManifest.xml: android:allowBackup="true" ← INSECURE
```

---

## Summary Table

| Mechanism | Protection | Bypass Method |
|-----------|-----------|---------------|
| UID Isolation | File access control | Root access |
| Process Isolation | Memory protection | Kernel exploit |
| SELinux | Mandatory access control | SELinux policy bypass |
| Seccomp | System call restriction | Seccomp bypass |
| File Permissions | Data directory access | Root, world-readable |

---

## Revision Questions

1. How does the Android sandbox use Linux UIDs for isolation?
2. What role does SELinux play in the Android sandbox?
3. What are five ways an attacker can escape the sandbox?
4. Why is external storage a security concern for sandboxed apps?
5. How do you verify sandbox integrity during a security test?

---

*Previous: [01-android-architecture.md](01-android-architecture.md) | Next: [03-permissions-model.md](03-permissions-model.md)*

---

*[Back to README](../README.md)*

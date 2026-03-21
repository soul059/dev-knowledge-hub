# Unit 5: iOS Security Architecture — Topic 5: Data Protection API

## Overview

The **iOS Data Protection API** provides file-level encryption using a hierarchy of keys derived from the device passcode and hardware UID. Every file on iOS is encrypted with a unique per-file key, and the availability of that key depends on the Data Protection class assigned to the file. Understanding this system is critical for assessing whether app data is properly protected at rest.

---

## 1. Data Protection Key Hierarchy

```
DATA PROTECTION KEY HIERARCHY:

┌─────────────────────────────────────┐
│        HARDWARE UID KEY             │
│    (Fused in Secure Enclave)        │
│    Unique per device, unreadable    │
└──────────────┬──────────────────────┘
               │ derives
┌──────────────▼──────────────────────┐
│        DEVICE KEY                    │
│    (UID + entangled with passcode)  │
└──────────────┬──────────────────────┘
               │ unwraps
┌──────────────▼──────────────────────┐
│        CLASS KEYS                    │
│  One per Data Protection class:     │
│  • Complete Protection              │
│  • Protected Unless Open            │
│  • Protected Until First Auth       │
│  • No Protection                    │
└──────────────┬──────────────────────┘
               │ unwraps
┌──────────────▼──────────────────────┐
│        PER-FILE KEYS                │
│  Unique AES-256 key per file        │
│  Generated when file is created     │
│  Wrapped with class key             │
└──────────────┬──────────────────────┘
               │ encrypts
┌──────────────▼──────────────────────┐
│        FILE CONTENT                  │
│  AES-256-XTS encryption             │
│  Hardware AES engine                 │
└─────────────────────────────────────┘

REMOTE WIPE:
  → Erases "effaceable storage" containing class keys
  → All file keys become unrecoverable
  → Entire device data destroyed instantly
  → No need to overwrite every sector
```

---

## 2. Data Protection Classes

```
FILE PROTECTION CLASSES:

NSFileProtectionComplete (Class A):
  → File accessible ONLY when device is unlocked
  → Class key discarded 10 seconds after lock
  → Strongest protection
  → Use for: sensitive documents, databases
  → Cannot read/write when device is locked
  → Background apps lose access!

NSFileProtectionCompleteUnlessOpen (Class B):
  → File stays accessible if opened before lock
  → Uses asymmetric key pair
  → Can create new files while locked
  → Use for: email attachments downloading in background

NSFileProtectionCompleteUntilFirstUserAuthentication (Class C):
  → DEFAULT class for third-party apps!
  → Key available after first unlock until reboot
  → Stays accessible when device locked
  → Use for: data needed by background processes
  → WARNING: Weaker than complete protection

NSFileProtectionNone (Class D):
  → File always accessible
  → Only protected by device UID key
  → Weakest protection
  → Use for: non-sensitive cached data only
```

---

## 3. Testing Data Protection

```bash
# === CHECK FILE PROTECTION CLASSES ===

# On jailbroken device, check protection class:
# List files and their protection classes
find /var/mobile/Containers/Data/Application/<UUID>/ -type f \
    -exec ls -la {} \;

# Using FileDP tool (jailbroken)
filedp -f /var/mobile/Containers/Data/Application/<UUID>/Documents/

# Using objection
objection -g com.example.app explore
> env                    # Show app paths
> ios plist cat /path/to/file.plist
> !ls -la /path/to/Documents/

# Using Frida to check protection class
```

```javascript
// Frida: Monitor file protection class on creation
var NSFileManager = ObjC.classes.NSFileManager;
var fm = NSFileManager.defaultManager();

Interceptor.attach(ObjC.classes.NSFileManager['- createFileAtPath:contents:attributes:'].implementation, {
    onEnter: function(args) {
        var path = ObjC.Object(args[2]).toString();
        var attrs = ObjC.Object(args[4]);
        if (attrs) {
            var protection = attrs.objectForKey_("NSFileProtectionKey");
            console.log("[FileCreate] " + path);
            console.log("  Protection: " + (protection ? protection.toString() : "DEFAULT (UntilFirstAuth)"));
        }
    }
});
```

```
TESTING METHODOLOGY:

1. Install and use the app normally
2. Lock the device
3. Attempt to access app files via SSH/tool
4. Files with Complete protection should be inaccessible
5. Files with UntilFirstAuth ARE accessible when locked
6. Check: Are sensitive files using strong enough protection?

COMMON FINDINGS:
  □ Database with sensitive data using default (Class C)
  □ Downloaded documents using NSFileProtectionNone
  □ Cached API responses without protection
  □ Log files containing sensitive data, no protection
  □ Temporary files with credentials, no protection
```

---

## 4. Secure Implementation

```swift
// Setting file protection when creating
let fileManager = FileManager.default
let data = sensitiveData.data(using: .utf8)!
let path = documentsPath.appendingPathComponent("secure.dat")

// Create with Complete Protection
fileManager.createFile(
    atPath: path.path,
    contents: data,
    attributes: [.protectionKey: FileProtectionType.complete]
)

// Change protection class on existing file
try fileManager.setAttributes(
    [.protectionKey: FileProtectionType.complete],
    ofItemAtPath: path.path
)

// Core Data with file protection
let description = NSPersistentStoreDescription()
description.setOption(
    FileProtectionType.complete as NSObject,
    forKey: NSPersistentStoreFileProtectionKey
)

// SQLite with file protection
let dbPath = documentsPath.appendingPathComponent("app.db")
try fileManager.setAttributes(
    [.protectionKey: FileProtectionType.complete],
    ofItemAtPath: dbPath.path
)
```

---

## Summary Table

| Protection Class | When Available | Default? | Best For |
|-----------------|----------------|----------|----------|
| Complete (A) | Unlocked only | No | Sensitive data |
| CompleteUnlessOpen (B) | Open before lock | No | Background downloads |
| UntilFirstAuth (C) | After first unlock | YES | Background processes |
| None (D) | Always | No | Non-sensitive cache |

---

## Revision Questions

1. How does the iOS Data Protection key hierarchy work?
2. What are the four Data Protection classes and their differences?
3. What is the default protection class for third-party apps?
4. How do you test file protection classes on a jailbroken device?
5. Why is "Complete" protection incompatible with background processing?

---

*Previous: [04-keychain.md](04-keychain.md) | Next: [06-app-transport-security.md](06-app-transport-security.md)*

---

*[Back to README](../README.md)*

# Unit 3: Android Testing — Topic 5: Root Detection Bypass

## Overview

Many security-sensitive apps (banking, enterprise, DRM) implement **root detection** to refuse running on rooted devices. For security testing, bypassing root detection is often necessary to perform comprehensive analysis. Understanding how root detection works enables both implementing and bypassing these checks.

---

## 1. Common Root Detection Methods

```
ROOT DETECTION TECHNIQUES:

1. CHECK FOR SU BINARY:
   File.exists("/system/bin/su")
   File.exists("/system/xbin/su")
   File.exists("/sbin/su")
   File.exists("/data/local/bin/su")

2. CHECK FOR ROOT MANAGEMENT APPS:
   PackageManager.getPackageInfo("com.topjohnwu.magisk")
   PackageManager.getPackageInfo("eu.chainfire.supersu")
   PackageManager.getPackageInfo("com.koushikdutta.superuser")

3. CHECK BUILD TAGS:
   Build.TAGS.contains("test-keys")
   → Indicates non-release build (often rooted)

4. CHECK SYSTEM PROPERTIES:
   Runtime.exec("getprop ro.debuggable")
   Runtime.exec("getprop ro.secure")
   → Debuggable=1 or secure=0 indicates root

5. CHECK WRITABLE SYSTEM PARTITIONS:
   Try to write to /system
   Mount command shows /system as rw

6. CHECK FOR BUSYBOX:
   File.exists("/system/bin/busybox")
   → Common utility installed on rooted devices

7. SAFETYNET / PLAY INTEGRITY API:
   Server-side verification of device integrity
   Hardest to bypass (requires Magisk Hide + module tuning)

8. NATIVE CODE CHECKS:
   → Root detection in .so files (harder to bypass)
   → Check /proc/self/maps for Frida/Xposed
   → Check for Frida server port (27042)
```

---

## 2. Bypass Methods

```bash
# === METHOD 1: MAGISK HIDE / ZYGISK DENYLIST ===
# Magisk Manager → Settings → Enable Zygisk
# Magisk Manager → Settings → Configure DenyList
# Add target app to DenyList
# → Hides Magisk from the app

# === METHOD 2: FRIDA SCRIPT ===
# frida -U -f com.example.app -l root-bypass.js --no-pause
```

```javascript
// Frida root detection bypass
Java.perform(function() {
    // Bypass File.exists() checks
    var File = Java.use("java.io.File");
    File.exists.implementation = function() {
        var path = this.getAbsolutePath();
        var rootPaths = ["/system/bin/su", "/system/xbin/su", "/sbin/su",
                        "/data/local/bin/su", "/system/bin/busybox"];
        
        for (var i = 0; i < rootPaths.length; i++) {
            if (path === rootPaths[i]) {
                console.log("[*] Bypassing root check for: " + path);
                return false;
            }
        }
        return this.exists();
    };
    
    // Bypass Build.TAGS check
    var Build = Java.use("android.os.Build");
    Build.TAGS.value = "release-keys";
    
    // Bypass Runtime.exec for root commands
    var Runtime = Java.use("java.lang.Runtime");
    Runtime.exec.overload('java.lang.String').implementation = function(cmd) {
        if (cmd.indexOf("su") !== -1 || cmd.indexOf("which") !== -1) {
            console.log("[*] Blocked exec: " + cmd);
            throw new Error("Command not found");
        }
        return this.exec(cmd);
    };
});
```

```bash
# === METHOD 3: OBJECTION ===
objection -g com.example.app explore
> android root disable
# Automatically patches common root detection methods

# === METHOD 4: REPACKAGE APK ===
# Decompile with apktool
# Find root detection code in smali
# Modify return values (return false instead of true)
# Rebuild and re-sign

# === METHOD 5: XPOSED MODULE ===
# RootCloak — Hides root from specific apps
# Requires Xposed Framework installed
```

---

## 3. Advanced Root Detection and Bypass

```
ADVANCED DETECTIONS:

FRIDA DETECTION:
  → Check for frida-server process
  → Check port 27042 (default Frida port)
  → Scan /proc/self/maps for frida libraries
  → Check for frida-related named pipes

  BYPASS:
  → Rename frida-server binary
  → Use non-default port: frida-server -l 0.0.0.0:1337
  → Use frida-gadget (injected, no server)
  → Patch detection in smali code

NATIVE CODE DETECTION:
  → Root checks in .so files (C/C++)
  → Harder to bypass than Java checks
  → Requires: Frida native hooks or binary patching

  BYPASS:
  → Frida Interceptor.attach on native functions
  → Binary patch .so file (change instructions)
  → Use Ghidra/IDA to find and patch checks

PLAY INTEGRITY API:
  → Server-side device integrity check
  → Cannot be bypassed client-side alone
  → Magisk + modules may pass basic checks
  → Hardware attestation: very difficult to bypass
```

---

## Summary Table

| Detection Method | Bypass | Difficulty |
|-----------------|--------|-----------|
| su binary check | Magisk Hide, Frida | Easy |
| Package check | Magisk Hide, Frida | Easy |
| Build tags | Frida hook | Easy |
| System properties | Frida hook | Easy |
| SafetyNet/Play Integrity | Magisk + modules | Medium-Hard |
| Native code checks | Frida native hooks | Hard |
| Frida detection | Rename/reconfig Frida | Medium |

---

## Revision Questions

1. List five common root detection techniques apps use.
2. How does Magisk Hide/Zygisk DenyList bypass root detection?
3. Write a Frida script to bypass File.exists() root checks.
4. What makes Play Integrity API harder to bypass than local checks?
5. How do you bypass root detection implemented in native code?

---

*Previous: [04-traffic-interception.md](04-traffic-interception.md) | Next: [06-tools-jadx-apktool-frida.md](06-tools-jadx-apktool-frida.md)*

---

*[Back to README](../README.md)*

# Unit 4: Android Vulnerabilities — Topic 4: Code Tampering

## Overview

**Code tampering** involves modifying a mobile application's code, resources, or behavior to bypass security controls, unlock premium features, inject malicious functionality, or cheat in games. Android apps are particularly vulnerable because APKs can be easily decompiled, modified, and repackaged.

---

## 1. Tampering Techniques

```
CODE TAMPERING METHODS:

1. APK REPACKAGING:
   apktool d app.apk → Modify smali/resources → apktool b → Re-sign
   → Change app logic, remove protections
   → Distribute modified version

2. RUNTIME MODIFICATION:
   → Frida hooks to change function behavior
   → Xposed modules to intercept calls
   → No permanent modification needed

3. BINARY PATCHING:
   → Modify DEX bytecode directly
   → Patch native .so libraries
   → Change specific instructions

4. RESOURCE MODIFICATION:
   → Modify config files, URLs
   → Change API endpoints
   → Alter certificate pins

5. DYNAMIC CODE LOADING HIJACK:
   → Replace dynamically loaded DEX/JAR files
   → Intercept class loading
   → Inject custom code via DexClassLoader
```

---

## 2. Anti-Tampering Mechanisms

```
TAMPER DETECTION APPROACHES:

1. APK SIGNATURE VERIFICATION:
   → App checks its own signature at runtime
   → Compare against known-good signature hash
   → If signature differs → app was repackaged

   // Check signature
   PackageInfo pInfo = getPackageManager().getPackageInfo(getPackageName(), PackageManager.GET_SIGNATURES);
   String signature = pInfo.signatures[0].toCharsString();
   if (!signature.equals(EXPECTED_SIGNATURE)) {
       // Tampered! Exit or restrict functionality
   }

2. CHECKSUM VERIFICATION:
   → Calculate hash of classes.dex at runtime
   → Compare against embedded known-good hash
   → Detects DEX modification

3. DEBUGGER DETECTION:
   → Check android.os.Debug.isDebuggerConnected()
   → Check /proc/self/status for TracerPid
   → Detect Frida server process
   → Check for Xposed framework

4. EMULATOR DETECTION:
   → Check Build.FINGERPRINT for "generic"
   → Check for emulator-specific files
   → Check sensor data (emulators have no real sensors)

5. INTEGRITY CHECKS IN NATIVE CODE:
   → Implement checks in C/C++ (.so files)
   → Harder to reverse than Java
   → Use obfuscation on native code too
```

---

## 3. Bypassing Anti-Tampering

```
BYPASS STRATEGIES:

SIGNATURE CHECK BYPASS:
  → Find the signature check method in smali
  → Modify to always return valid
  → Or hook with Frida to return expected signature

  Java.perform(function() {
      var PackageManager = Java.use("android.app.ApplicationPackageManager");
      PackageManager.getPackageInfo.overload('java.lang.String', 'int')
          .implementation = function(pkg, flags) {
              // Return original package info with expected signature
              return this.getPackageInfo(pkg, flags);
          };
  });

CHECKSUM BYPASS:
  → Find checksum comparison code
  → Patch comparison to always succeed
  → Or update embedded hash after modification

DEBUGGER DETECTION BYPASS:
  → Frida: Hook isDebuggerConnected() → return false
  → Patch TracerPid check in /proc/self/status
```

---

## 4. Commercial Protection Solutions

```
ANTI-TAMPERING PRODUCTS:

PROGUARD / R8 (Free, built-in):
  → Code obfuscation (rename classes/methods)
  → Dead code removal
  → Basic protection only
  → Included in Android build tools

DEXGUARD (Commercial):
  → Advanced obfuscation
  → String encryption
  → Class encryption
  → Anti-debugging
  → Anti-tampering
  → Root detection

ARXAN / DIGITAL.AI (Commercial):
  → Code hardening
  → RASP (Runtime Application Self-Protection)
  → Integrity verification
  → Anti-reverse engineering

GUARDSQUARE (Commercial):
  → DexGuard (Android) + iXGuard (iOS)
  → Multiple layers of protection
  → Industry standard for financial apps
```

---

## Summary Table

| Protection | Bypass Method | Difficulty |
|-----------|---------------|-----------|
| Signature check | Frida hook / smali patch | Easy-Medium |
| Checksum verify | Update hash / patch check | Medium |
| Debugger detect | Frida hook | Easy |
| Root detection | Magisk Hide / Frida | Easy-Medium |
| ProGuard obfuscation | jadx still readable | Easy |
| DexGuard encryption | Advanced tools needed | Hard |
| Native checks | Binary patching | Medium-Hard |

---

## Revision Questions

1. What are the main code tampering techniques for Android?
2. How does APK signature verification detect tampering?
3. What is the difference between ProGuard and DexGuard?
4. How do you bypass signature verification using Frida?
5. Why are native code anti-tampering checks more resistant?

---

*Previous: [03-insecure-authentication.md](03-insecure-authentication.md) | Next: [05-reverse-engineering.md](05-reverse-engineering.md)*

---

*[Back to README](../README.md)*

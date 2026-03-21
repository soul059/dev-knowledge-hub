# Unit 4: Android Vulnerabilities — Topic 5: Reverse Engineering

## Overview

**Reverse engineering** Android apps means analyzing the compiled application to understand its functionality, extract secrets, find vulnerabilities, and bypass protections — without access to the original source code. Since Android apps are distributed as APKs containing DEX bytecode, they are relatively easy to decompile compared to native applications.

---

## 1. Reverse Engineering Workflow

```
REVERSE ENGINEERING PROCESS:

1. OBTAIN APK
   └── Pull from device, download from mirror

2. INITIAL RECONNAISSANCE
   ├── APK file size, libraries, architecture
   ├── AndroidManifest.xml review
   └── Third-party SDK identification

3. DECOMPILATION
   ├── jadx → Java source (best readability)
   ├── apktool → Smali + resources
   ├── dex2jar → JAR → JD-GUI
   └── IDA Pro / Ghidra → Native libraries

4. CODE ANALYSIS
   ├── Identify authentication flow
   ├── Find encryption/decryption routines
   ├── Locate API endpoints and keys
   ├── Map business logic
   └── Find security controls (root detect, pinning)

5. DYNAMIC ANALYSIS
   ├── Frida for runtime hooking
   ├── Debug with Android Studio
   └── Network traffic analysis

6. DOCUMENTATION
   └── Document findings, attack paths
```

---

## 2. Decompilation Tools

```
DEX → JAVA (Readable code):
  jadx target.apk -d output/
  → Best Java readability
  → Handles obfuscated code with --deobf
  → GUI with search and navigation

DEX → SMALI (Assembly-level):
  apktool d target.apk
  → Dalvik assembly language
  → Preserves exact bytecode logic
  → Required for modification

NATIVE CODE (.so files):
  Ghidra (Free, NSA):
  → Supports ARM, x86
  → Decompiler to C-like pseudocode
  → Cross-references and graphs
  → Script support (Java/Python)

  IDA Pro (Commercial):
  → Industry standard disassembler
  → Best ARM support
  → Hex-Rays decompiler (paid)
  → Plugin ecosystem

  radare2 / rizin (Free, CLI):
  → Command-line disassembler
  → Scriptable analysis
  → r2frida integration
```

---

## 3. Anti-Reverse Engineering Techniques

```
PROTECTIONS AND THEIR EFFECTIVENESS:

CODE OBFUSCATION (ProGuard/R8):
  → Renames classes: LoginActivity → a.b.c
  → Renames methods: validatePassword → a()
  → Removes unused code
  → EFFECTIVENESS: Low — logic still readable

STRING ENCRYPTION:
  → Strings decoded at runtime
  → "password" → decrypt(byte[]{0x70, 0x61, ...})
  → EFFECTIVENESS: Medium — Frida can dump decrypted strings

  // Frida: Dump all decrypted strings
  Java.perform(function() {
      var StringDecryptor = Java.use("com.example.StringDecryptor");
      StringDecryptor.decrypt.implementation = function(input) {
          var result = this.decrypt(input);
          console.log("[String] " + result);
          return result;
      };
  });

CONTROL FLOW OBFUSCATION:
  → Inserts fake branches, opaque predicates
  → Flattens control flow
  → EFFECTIVENESS: Medium-High

NATIVE CODE MIGRATION:
  → Move sensitive logic to C/C++ (.so files)
  → Harder to decompile than Java
  → EFFECTIVENESS: Medium-High

PACKING/ENCRYPTION:
  → Encrypt DEX, decrypt at runtime
  → Custom classloaders
  → DexProtector, Bangcle
  → EFFECTIVENESS: High (requires runtime dump)

  // Bypass: Dump DEX from memory at runtime
  // Frida script to dump loaded DEX files
```

---

## 4. Practical Reverse Engineering Tips

```
FINDING INTERESTING CODE:

ENTRY POINTS:
  → AndroidManifest.xml → main Activity (LAUNCHER)
  → Search for: "login", "auth", "password", "encrypt"
  → Trace from UI buttons back to logic

API ENDPOINTS:
  → Search for "http", "https", "/api/", "endpoint"
  → Find Retrofit/OkHttp service definitions
  → Map all API calls

ENCRYPTION:
  → Search for "Cipher", "SecretKey", "AES", "encrypt"
  → Find key derivation and storage
  → Identify algorithm and mode

AUTHENTICATION:
  → Find login Activity/Fragment
  → Trace credential handling
  → Find token storage location
  → Map session management

INTERESTING CLASSES:
  → *Utils, *Helper, *Manager, *Service
  → *Crypto*, *Security*, *Auth*
  → BuildConfig (contains build constants)
  → R class (resource IDs)
```

---

## Summary Table

| Tool | Input | Output | Best For |
|------|-------|--------|----------|
| jadx | APK/DEX | Java source | Code review |
| apktool | APK | Smali + resources | Modification |
| Ghidra | .so (native) | C pseudocode | Native analysis |
| Frida | Running app | Runtime data | Dynamic RE |
| strings | Any binary | ASCII strings | Quick recon |

---

## Revision Questions

1. Describe the Android reverse engineering workflow.
2. What tools are used for decompiling DEX vs native code?
3. How does code obfuscation (ProGuard) protect apps and what are its limits?
4. How do you use Frida to dump decrypted strings at runtime?
5. What makes native code harder to reverse engineer than Java?

---

*Previous: [04-code-tampering.md](04-code-tampering.md) | Next: [06-intent-vulnerabilities.md](06-intent-vulnerabilities.md)*

---

*[Back to README](../README.md)*

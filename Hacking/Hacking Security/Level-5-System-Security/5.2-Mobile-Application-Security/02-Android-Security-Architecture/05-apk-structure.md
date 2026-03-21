# Unit 2: Android Security Architecture — Topic 5: APK Structure

## Overview

An **APK (Android Package)** is the file format used to distribute and install Android applications. Understanding APK structure is essential for mobile security testing because it reveals the app's code, resources, permissions, and configuration — all of which are targets for security analysis.

---

## 1. APK File Structure

```
APK FILE (ZIP archive):

example.apk (ZIP)
├── AndroidManifest.xml      ← App configuration (binary XML)
├── classes.dex              ← Compiled Java/Kotlin bytecode
├── classes2.dex             ← Additional DEX (if multidex)
├── resources.arsc           ← Compiled resources
├── res/                     ← Resource files
│   ├── layout/              ← UI layouts
│   ├── values/              ← Strings, colors, styles
│   ├── drawable/            ← Images
│   ├── xml/                 ← XML configurations
│   └── raw/                 ← Raw files
├── assets/                  ← Raw asset files (not compiled)
├── lib/                     ← Native libraries (.so files)
│   ├── armeabi-v7a/         ← ARM 32-bit
│   ├── arm64-v8a/           ← ARM 64-bit
│   ├── x86/                 ← Intel 32-bit
│   └── x86_64/              ← Intel 64-bit
├── META-INF/                ← Signing information
│   ├── MANIFEST.MF          ← File hashes
│   ├── CERT.SF              ← Signed file hashes
│   └── CERT.RSA             ← Certificate
└── kotlin/                  ← Kotlin metadata (if Kotlin)
```

---

## 2. Key Files for Security Analysis

```
AndroidManifest.xml:
  THE MOST IMPORTANT FILE FOR SECURITY REVIEW
  Contains:
  ├── Package name (unique identifier)
  ├── Permissions requested (<uses-permission>)
  ├── Components (activities, services, receivers, providers)
  ├── Exported components (accessible by other apps)
  ├── Intent filters (how components are invoked)
  ├── minSdkVersion / targetSdkVersion
  ├── Backup configuration (android:allowBackup)
  ├── Debuggable flag (android:debuggable)
  ├── Network security config reference
  └── Custom permissions defined

classes.dex:
  → Dalvik Executable bytecode
  → Contains ALL Java/Kotlin application code
  → Can be decompiled to readable Java with jadx
  → Search for: hardcoded strings, API keys, crypto keys
  → Contains business logic, authentication code

lib/ (Native Libraries):
  → .so files (shared objects — compiled C/C++)
  → Harder to reverse engineer than DEX
  → May contain crypto routines, anti-tampering
  → Analyze with: IDA Pro, Ghidra, radare2

assets/:
  → Configuration files, databases, certificates
  → May contain hardcoded credentials
  → Custom file formats, encrypted data
  → Web content for WebView-based apps

res/xml/network_security_config.xml:
  → Network security configuration
  → Certificate pinning definitions
  → Cleartext traffic permissions
```

---

## 3. Extracting and Analyzing APKs

```bash
# METHOD 1: Pull APK from device
adb shell pm path com.example.app
# /data/app/com.example.app-1/base.apk
adb pull /data/app/com.example.app-1/base.apk

# METHOD 2: Download from APK mirror sites
# (apkpure.com, apkmirror.com)

# DECOMPILE WITH APKTOOL (resources + smali)
apktool d example.apk -o output_dir/
# Produces: decoded AndroidManifest.xml, smali code, resources

# DECOMPILE WITH JADX (Java source)
jadx example.apk -d output_dir/
# Produces: readable Java source code

# QUICK ANALYSIS — Unzip as ZIP
unzip example.apk -d extracted/

# STRINGS ANALYSIS
strings classes.dex | grep -i "password\|secret\|api_key\|token\|http"

# CHECK FOR SENSITIVE FILES
find output_dir/ -name "*.db" -o -name "*.sqlite" -o -name "*.key" -o -name "*.pem" -o -name "*.p12"
```

---

## 4. Security Findings in APK

```
COMMON SECURITY FINDINGS:

MANIFEST ISSUES:
  android:debuggable="true"      ← Debug access in production!
  android:allowBackup="true"     ← Data extractable via backup
  android:exported="true"        ← Component accessible by any app
  android:usesCleartextTraffic="true" ← HTTP allowed

HARDCODED SECRETS (in classes.dex):
  API keys, AWS credentials
  Firebase URLs and secrets
  Encryption keys
  OAuth client secrets
  Backend server URLs

INSECURE LIBRARIES:
  → Outdated libraries with known CVEs
  → Check: library versions in build metadata
  → Tools: OWASP Dependency-Check, retire.js

NATIVE CODE ISSUES:
  → Buffer overflows in .so files
  → Hardcoded keys in native code
  → Missing ASLR/stack protections
  → Weak crypto implementations
```

---

## Summary Table

| File/Directory | Contents | Security Relevance |
|---------------|----------|-------------------|
| AndroidManifest.xml | App configuration | Permissions, exported components |
| classes.dex | Java bytecode | Hardcoded secrets, business logic |
| lib/*.so | Native code | Buffer overflows, native crypto |
| assets/ | Raw files | Config files, certs, databases |
| res/xml/ | Network config | Certificate pinning, TLS config |
| META-INF/ | Signatures | Certificate verification |

---

## Revision Questions

1. What are the main files in an APK and their purposes?
2. How do you extract and decompile an APK for analysis?
3. What security-relevant information is in AndroidManifest.xml?
4. What should you search for in decompiled classes.dex?
5. Why are native libraries (.so) harder to analyze than DEX code?

---

*Previous: [04-android-security-features.md](04-android-security-features.md) | Next: [06-signing-and-verification.md](06-signing-and-verification.md)*

---

*[Back to README](../README.md)*

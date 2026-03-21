# Unit 6: iOS Testing — Topic 2: Static Analysis

## Overview

**Static analysis** of iOS applications involves examining the app binary, resources, and configuration files without executing the application. This reveals hardcoded credentials, insecure configurations, embedded URLs, cryptographic implementations, and potential vulnerabilities — all from the IPA file alone.

---

## 1. Obtaining the IPA File

```bash
# === FROM APP STORE (macOS) ===
# Use Apple Configurator 2 or ipatool
brew install ipatool
ipatool auth login --email user@example.com
ipatool download --bundle-identifier com.example.app

# === FROM JAILBROKEN DEVICE ===
# Find app path
ssh root@device 'find /var/containers/Bundle/Application -name "*.app"'

# Copy app bundle
scp -r root@device:/var/containers/Bundle/Application/<UUID>/App.app ./

# Using Clutch (decrypt App Store encryption)
ssh root@device
Clutch -d com.example.app
# Outputs decrypted IPA

# Using frida-ios-dump (decrypt + dump)
pip3 install frida-ios-dump
dump.py com.example.app
# Creates decrypted IPA file

# === FROM BACKUP ===
# iTunes backup contains app data (not binary)
# Use idevicebackup2 for extraction
idevicebackup2 backup --full ./backup/
```

---

## 2. IPA Structure Analysis

```
IPA FILE STRUCTURE:

app.ipa (ZIP archive)
├── Payload/
│   └── App.app/                    ← App bundle
│       ├── App                      ← Main binary (Mach-O)
│       ├── Info.plist               ← App configuration
│       ├── embedded.mobileprovision ← Provisioning profile
│       ├── _CodeSignature/          ← Code signature
│       ├── Frameworks/              ← Embedded frameworks
│       ├── Assets.car               ← Compiled assets
│       ├── *.storyboardc            ← Compiled storyboards
│       ├── *.nib                    ← Interface files
│       └── *.js, *.html             ← Web resources
└── iTunesMetadata.plist             ← Store metadata

ANALYSIS COMMANDS:
# Extract IPA
unzip app.ipa -d extracted/

# View Info.plist
plutil -p extracted/Payload/App.app/Info.plist

# Check binary architecture
file extracted/Payload/App.app/App
lipo -info extracted/Payload/App.app/App

# List embedded frameworks
ls extracted/Payload/App.app/Frameworks/

# Check provisioning profile
security cms -D -i extracted/Payload/App.app/embedded.mobileprovision

# Verify code signature
codesign -dvvv extracted/Payload/App.app/
```

---

## 3. Binary Analysis

```bash
# === STRING EXTRACTION ===
strings extracted/Payload/App.app/App | grep -i "password\|secret\|key\|token\|api"
strings extracted/Payload/App.app/App | grep -E "https?://"
strings extracted/Payload/App.app/App | grep -i "firebase\|aws\|azure"

# === OBJECTIVE-C CLASS DUMP ===
# Extract class headers (ObjC apps)
class-dump extracted/Payload/App.app/App > headers.h
# Review: class names, method signatures, properties

# For Swift apps (limited class-dump support)
# Use Ghidra or dsdump instead
dsdump --objc extracted/Payload/App.app/App

# === GHIDRA ANALYSIS ===
# Import Mach-O binary
# Analyze → Auto Analysis
# Look for:
#   → Encryption functions (CCCrypt, AES, RSA)
#   → Network calls (NSURLSession, URLRequest)
#   → Keychain operations (SecItem*)
#   → File operations with protection class
#   → Jailbreak detection strings

# === otool (macOS built-in) ===
# List shared libraries
otool -L extracted/Payload/App.app/App

# Show load commands
otool -l extracted/Payload/App.app/App

# Check PIE (Position Independent Executable)
otool -hv extracted/Payload/App.app/App
# Look for: PIE flag in header

# Check for ARC (Automatic Reference Counting)
otool -Iv extracted/Payload/App.app/App | grep -i "objc_release\|objc_retain"
```

---

## 4. Configuration Review

```
INFO.PLIST CHECKS:

SECURITY-RELEVANT KEYS:
  NSAppTransportSecurity    → ATS configuration
  CFBundleURLTypes          → Registered URL schemes
  NSCameraUsageDescription  → Camera access justification
  NSLocationUsageDescription → Location access
  UIRequiredDeviceCapabilities → Required hardware
  LSApplicationQueriesSchemes → Apps it can query
  UIBackgroundModes         → Background capabilities

WHAT TO LOOK FOR:
  □ ATS exceptions (NSAllowsArbitraryLoads)
  □ Custom URL schemes (potential deep link abuse)
  □ Excessive permissions requested
  □ Background modes (location, fetch, etc.)
  □ get-task-allow = true (debug enabled!)

PROVISIONING PROFILE REVIEW:
  □ Team ID and developer info
  □ Entitlements granted
  □ Device UDIDs (Ad Hoc)
  □ Expiration date
  □ get-task-allow value
```

---

## 5. Automated Static Analysis

```
MobSF (Mobile Security Framework):

# Install and run
docker pull opensecurity/mobile-security-framework-mobsf
docker run -it -p 8000:8000 opensecurity/mobile-security-framework-mobsf

# Upload IPA → Automated analysis
# Reports include:
#   → Binary analysis (PIE, ARC, stack canaries)
#   → Info.plist review
#   → ATS configuration
#   → Hardcoded strings and URLs
#   → Third-party library identification
#   → Permission analysis
#   → Certificate information
#   → Insecure API usage

OTHER TOOLS:
  → iLEAPP (iOS Logs, Events, and Properties Parser)
  → Needle (iOS security testing — deprecated)
  → Passionfruit (iOS app analyzer)
```

---

## Summary Table

| Analysis Area | Tools | Key Findings |
|--------------|-------|-------------|
| Binary strings | strings, rabin2 | Hardcoded secrets, URLs |
| Class dump | class-dump, dsdump | API surface, methods |
| Config review | plutil, plistutil | ATS, permissions |
| Binary security | otool, rabin2 | PIE, ARC, stack canaries |
| Disassembly | Ghidra, IDA Pro | Crypto, auth logic |
| Automated | MobSF | Comprehensive report |

---

## Revision Questions

1. How do you obtain and decrypt an IPA file from a device?
2. What is the structure of an IPA file?
3. What security-relevant keys should be checked in Info.plist?
4. How do you perform binary analysis with class-dump and otool?
5. What automated tools are available for iOS static analysis?

---

*Previous: [01-environment-setup.md](01-environment-setup.md) | Next: [03-dynamic-analysis.md](03-dynamic-analysis.md)*

---

*[Back to README](../README.md)*

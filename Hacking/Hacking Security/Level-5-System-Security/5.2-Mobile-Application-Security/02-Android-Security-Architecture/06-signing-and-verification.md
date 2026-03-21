# Unit 2: Android Security Architecture — Topic 6: Signing and Verification

## Overview

Android uses **code signing** to verify the integrity and authenticity of applications. Every APK must be digitally signed before installation. The signing mechanism ensures that app updates come from the same developer, enables permission sharing between apps from the same developer, and prevents tampering with the application package.

---

## 1. APK Signing Schemes

```
ANDROID SIGNING EVOLUTION:

v1 SIGNING (JAR Signing):
  → Signs individual files in APK
  → Based on Java JAR signing
  → META-INF/MANIFEST.MF: SHA-256 hash of each file
  → META-INF/CERT.SF: Signed hashes
  → META-INF/CERT.RSA: Certificate + signature
  → Weakness: Files outside META-INF can be added without invalidating signature

v2 SIGNING (APK Signature Scheme v2, Android 7.0+):
  → Signs the ENTIRE APK file
  → Signature block inserted before Central Directory
  → Protects all contents including META-INF
  → Faster verification (whole-file check)
  → More secure than v1

v3 SIGNING (APK Signature Scheme v3, Android 9.0+):
  → Supports key rotation
  → Can update signing key while maintaining trust chain
  → Backward compatible with v2
  → Proof-of-rotation structure

v4 SIGNING (Android 11+):
  → Supports incremental APK installation
  → Merkle tree-based verification
  → Used with ADB incremental install
```

---

## 2. Signing Process

```
SIGNING FLOW:

Developer:
  ┌──────────┐    sign     ┌──────────┐
  │ Unsigned │──────────▶│  Signed  │
  │   APK    │            │   APK    │
  └──────────┘            └──────────┘
       │                       │
       │    Using keystore:    │
       │    ┌──────────────┐   │
       └───▶│ Private Key  │───┘
            │ Certificate  │
            └──────────────┘

# Sign with apksigner (Android SDK)
apksigner sign --ks my-keystore.jks --ks-key-alias my-key example.apk

# Verify signature
apksigner verify --verbose example.apk

# Check signing certificate details
apksigner verify --print-certs example.apk
# Outputs: Signer #1 certificate DN, SHA-256, SHA-1

# Legacy jarsigner (v1 only)
jarsigner -verify -verbose -certs example.apk

# keytool — View keystore contents
keytool -list -v -keystore my-keystore.jks
```

---

## 3. Security Implications

```
WHAT SIGNING PROTECTS:
  ✓ Integrity — Detects modification of APK contents
  ✓ Authenticity — Updates must use same key
  ✓ Trust — Apps from same developer share identity
  
WHAT SIGNING DOES NOT PROTECT:
  ✗ Does NOT verify who the developer is (self-signed)
  ✗ Does NOT prevent reverse engineering
  ✗ Does NOT prevent repackaging with different key
  ✗ Does NOT protect against runtime modification

REPACKAGING ATTACK:
  1. Download legitimate APK
  2. Decompile with apktool
  3. Modify code (inject malware)
  4. Rebuild APK
  5. Sign with attacker's key
  6. Distribute modified APK
  → Different signature, but Android installs it as new app
  → Side-loaded apps bypass Play Store checks

DETECTION:
  → Compare signature hash with known-good
  → App can check its own signature at runtime
  → Play Integrity API verifies app authenticity
  → Google Play App Signing manages keys server-side
```

---

## 4. Testing Signing

```bash
# Extract certificate from APK
apksigner verify --print-certs app.apk

# Check if APK uses v1, v2, v3 signing
apksigner verify -v app.apk
# Verified using v1 scheme (JAR signing): true
# Verified using v2 scheme (APK Signature Scheme v2): true
# Verified using v3 scheme (APK Signature Scheme v3): false

# Check for debug certificate (indicates test build)
keytool -printcert -jarfile app.apk
# Debug certs typically have:
# CN=Android Debug, O=Android, C=US

# Repackage and re-sign (for testing)
apktool d app.apk -o app_decoded
# ... make modifications ...
apktool b app_decoded -o app_modified.apk
# Generate test key
keytool -genkey -v -keystore test.jks -alias test -keyalg RSA -keysize 2048 -validity 10000
# Sign modified APK
apksigner sign --ks test.jks --ks-key-alias test app_modified.apk
# Verify
apksigner verify app_modified.apk
```

---

## Summary Table

| Scheme | Version | Scope | Key Feature |
|--------|---------|-------|-------------|
| v1 | All | Per-file | JAR signing (legacy) |
| v2 | 7.0+ | Whole APK | Full APK protection |
| v3 | 9.0+ | Whole APK | Key rotation support |
| v4 | 11+ | Incremental | Merkle tree verification |

---

## Revision Questions

1. What are the four APK signing schemes and their differences?
2. What security does APK signing provide and not provide?
3. How does a repackaging attack work?
4. How do you verify an APK's signing certificate?
5. What is the significance of a debug certificate in a production APK?

---

*Previous: [05-apk-structure.md](05-apk-structure.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

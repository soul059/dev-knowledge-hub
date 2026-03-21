# Unit 3: Android Testing — Topic 4: Traffic Interception

## Overview

**Traffic interception** is critical for analyzing how a mobile app communicates with its backend servers. By setting up a proxy between the app and its API, testers can inspect requests, modify parameters, test authentication, and identify vulnerabilities in the communication layer. Android's evolving security model makes this increasingly challenging.

---

## 1. Proxy Configuration

```
TRAFFIC INTERCEPTION SETUP:

┌──────────┐    WiFi Proxy    ┌──────────┐    HTTPS    ┌──────────┐
│ Android  │─────────────────▶│  Burp    │────────────▶│  API     │
│ Device   │◀─────────────────│  Suite   │◀────────────│  Server  │
└──────────┘                  └──────────┘             └──────────┘
     │                             │
     │  Burp CA cert installed     │ Decrypts HTTPS
     │  on device                  │ Re-encrypts
     └─────────────────────────────┘

STEPS:
1. Configure Burp to listen on all interfaces (0.0.0.0:8080)
2. Connect device to same WiFi network
3. Set device proxy: Settings → WiFi → Modify → Manual Proxy
   Host: <burp_machine_IP>  Port: 8080
4. Install Burp CA certificate on device
5. Verify interception working: visit http://burp in device browser
```

---

## 2. Certificate Installation (Android 7+)

```bash
# Android 7+ apps targeting SDK 24+ only trust SYSTEM certificates
# User certificates are NOT trusted by default!

# === METHOD 1: Install as System Cert (Requires Root) ===

# Export Burp cert in DER format
# Burp → Proxy → Options → Import/Export CA → Export Certificate in DER format

# Convert to PEM
openssl x509 -inform DER -in burp-cert.der -out burp-cert.pem

# Get hash for filename
openssl x509 -inform PEM -subject_hash_old -in burp-cert.pem | head -1
# Output: 9a5ba575

# Push to system cert store
adb root
adb remount
adb push burp-cert.pem /system/etc/security/cacerts/9a5ba575.0
adb shell chmod 644 /system/etc/security/cacerts/9a5ba575.0
adb reboot

# === METHOD 2: Modify Network Security Config (Repackage) ===
# Decompile APK, add to AndroidManifest.xml:
# android:networkSecurityConfig="@xml/network_security_config"

# Create res/xml/network_security_config.xml:
# <network-security-config>
#   <base-config>
#     <trust-anchors>
#       <certificates src="system" />
#       <certificates src="user" />  ← Trust user certs
#     </trust-anchors>
#   </base-config>
# </network-security-config>

# Rebuild and re-sign APK

# === METHOD 3: Magisk Module ===
# Use MagiskTrustUserCerts module
# Automatically moves user certs to system trust store
```

---

## 3. Certificate Pinning Bypass

```
CERTIFICATE PINNING:
  App only trusts specific server certificates
  Rejects proxy certificates → Interception fails!

BYPASS METHODS:

1. FRIDA SCRIPT:
   frida -U -f com.example.app -l ssl-pinning-bypass.js --no-pause
   → Universal SSL pinning bypass scripts available
   → Hooks TrustManager, OkHttp, custom pinning

2. OBJECTION:
   objection -g com.example.app explore
   > android sslpinning disable
   → Automatically patches common pinning libraries

3. APKTOOL REPACKAGE:
   → Decompile, modify network security config
   → Remove pinning code from smali
   → Rebuild and re-sign

4. XPOSED/MAGISK MODULES:
   → SSLUnpinning, TrustMeAlready
   → System-wide pinning bypass

5. SPECIFIC LIBRARY PATCHES:
   OkHttp: Hook CertificatePinner.check()
   Retrofit: Hook SSL factory
   Custom: Identify and hook verification method
```

---

## 4. Common Findings in Traffic Analysis

```
WHAT TO LOOK FOR:

AUTHENTICATION:
□ Credentials sent in plaintext
□ Weak token generation
□ Missing token expiration
□ Token in URL parameters (logged)
□ No session invalidation on logout

AUTHORIZATION:
□ IDOR (Insecure Direct Object Reference)
□ Missing server-side authorization checks
□ Horizontal/vertical privilege escalation
□ Role manipulation in requests

DATA EXPOSURE:
□ Sensitive data in responses (PII, financial)
□ Verbose error messages
□ Debug information in headers
□ Unnecessary data returned

API SECURITY:
□ Missing rate limiting
□ No input validation
□ SQL injection via API
□ Missing HTTPS enforcement
□ API versioning issues
```

---

## Summary Table

| Challenge | Solution | Difficulty |
|-----------|----------|-----------|
| HTTPS interception | Install proxy CA cert | Easy |
| SDK 24+ cert trust | System cert (root) or repackage | Medium |
| Certificate pinning | Frida/Objection bypass | Medium |
| Non-HTTP protocols | Wireshark, custom proxy | Hard |
| Certificate transparency | Time-limited bypass | Hard |

---

## Revision Questions

1. How do you set up traffic interception for an Android app?
2. Why doesn't a user-installed CA certificate work on Android 7+?
3. What are four methods to bypass certificate pinning?
4. What security issues should you look for in intercepted traffic?
5. How does Objection's SSL pinning bypass work?

---

*Previous: [03-dynamic-analysis.md](03-dynamic-analysis.md) | Next: [05-root-detection-bypass.md](05-root-detection-bypass.md)*

---

*[Back to README](../README.md)*

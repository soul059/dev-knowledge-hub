# Unit 3: Android Testing — Topic 2: Static Analysis

## Overview

**Static analysis** examines an Android application **without executing it**. By decompiling the APK and reviewing its code, configuration, and resources, testers can identify security vulnerabilities, hardcoded secrets, insecure configurations, and privacy concerns. Static analysis is typically the first step in mobile application security testing.

---

## 1. Static Analysis Workflow

```
STATIC ANALYSIS PROCESS:

1. OBTAIN APK
   ├── Pull from device (adb pull)
   ├── Download from app store/mirror
   └── Receive from client

2. AUTOMATED SCAN
   ├── MobSF upload and scan
   ├── Review findings report
   └── Identify areas for manual review

3. DECOMPILE
   ├── jadx → Java source code
   ├── apktool → Smali + resources
   └── dex2jar → JAR file

4. MANIFEST REVIEW
   ├── Permissions analysis
   ├── Exported components
   ├── Configuration flags
   └── Intent filters

5. CODE REVIEW
   ├── Hardcoded secrets
   ├── Crypto implementation
   ├── Authentication logic
   ├── Data handling
   └── Third-party libraries

6. DOCUMENT FINDINGS
```

---

## 2. Manifest Analysis

```xml
<!-- KEY SECURITY CHECKS IN AndroidManifest.xml -->

<!-- CHECK 1: Debuggable flag -->
<application android:debuggable="true" />
<!-- FINDING: Debug mode enabled in production! -->

<!-- CHECK 2: Backup configuration -->
<application android:allowBackup="true" />
<!-- FINDING: Data extractable via adb backup -->

<!-- CHECK 3: Cleartext traffic -->
<application android:usesCleartextTraffic="true" />
<!-- FINDING: HTTP connections allowed -->

<!-- CHECK 4: Exported components without protection -->
<activity android:name=".AdminActivity"
          android:exported="true" />
<!-- FINDING: Accessible by any app, no permission required -->

<!-- CHECK 5: Content providers -->
<provider android:name=".DataProvider"
          android:exported="true"
          android:grantUriPermissions="true" />
<!-- FINDING: Data accessible to all apps -->

<!-- CHECK 6: Excessive permissions -->
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.CAMERA" />
<!-- REVIEW: Are all permissions justified? -->

<!-- CHECK 7: Min SDK version -->
<uses-sdk android:minSdkVersion="16" android:targetSdkVersion="33" />
<!-- FINDING: minSdk 16 = Android 4.1, missing modern security features -->
```

---

## 3. Code Review Targets

```bash
# Search for hardcoded secrets
grep -rn "api_key\|apikey\|api-key\|secret\|password\|token" --include="*.java" output_dir/
grep -rn "AIza\|AKIA\|sk_live\|pk_live" --include="*.java" output_dir/  # Google/AWS/Stripe keys
grep -rn "firebase\|amazonaws\|azure" --include="*.java" output_dir/

# Search for insecure crypto
grep -rn "MD5\|SHA1\|DES\|RC4\|ECB" --include="*.java" output_dir/
grep -rn "SecretKeySpec\|Cipher.getInstance" --include="*.java" output_dir/

# Search for hardcoded encryption keys
grep -rn "AES\|encrypt\|decrypt\|SecretKey" --include="*.java" output_dir/

# Search for insecure HTTP
grep -rn "http://" --include="*.java" output_dir/

# Search for WebView issues
grep -rn "setJavaScriptEnabled\|addJavascriptInterface\|setAllowFileAccess" --include="*.java" output_dir/

# Search for logging sensitive data
grep -rn "Log\.\|println\|System\.out" --include="*.java" output_dir/

# Search for insecure storage
grep -rn "MODE_WORLD_READABLE\|MODE_WORLD_WRITEABLE\|getExternalStorage" --include="*.java" output_dir/

# Search for SQL injection
grep -rn "rawQuery\|execSQL\|query(" --include="*.java" output_dir/

# Search for certificate validation bypass
grep -rn "TrustAllCerts\|AllowAllHostnames\|ALLOW_ALL\|trustAllCerts\|X509TrustManager" --include="*.java" output_dir/
```

---

## 4. Automated Tools

```
MobSF (Mobile Security Framework):
  → Upload APK → Automated scan
  → Checks: manifest, code, libraries, crypto
  → Generates detailed report with severity ratings
  → Supports Android AND iOS
  → docker run -p 8000:8000 opensecurity/mobile-security-framework-mobsf

QARK (Quick Android Review Kit):
  → Python-based automated scanner
  → Focuses on common Android vulnerabilities
  → Generates report and proof-of-concept exploits

AndroBugs:
  → Python-based vulnerability scanner
  → Checks for common Android security issues
  → Generates detailed findings report

JADX + grep:
  → Decompile with jadx
  → Custom grep patterns for secrets
  → Most flexible approach
  → Combine with tools like trufflehog/gitleaks for secret detection
```

---

## Summary Table

| Check | What to Look For | Severity |
|-------|-----------------|----------|
| Debuggable | `android:debuggable="true"` | High |
| Backup | `android:allowBackup="true"` | Medium |
| Exported components | `android:exported="true"` without permission | High |
| Hardcoded secrets | API keys, passwords, tokens in code | Critical |
| Weak crypto | MD5, SHA1, DES, ECB mode, hardcoded keys | High |
| HTTP usage | `http://` URLs | High |
| WebView | JS enabled + file access | High |
| Logging | Sensitive data in Log.d/Log.e | Medium |

---

## Revision Questions

1. What is the static analysis workflow for Android apps?
2. What are the most critical AndroidManifest.xml security checks?
3. How do you search for hardcoded secrets in decompiled code?
4. What tools automate Android static analysis?
5. Why is checking the minSdkVersion important for security?

---

*Previous: [01-environment-setup.md](01-environment-setup.md) | Next: [03-dynamic-analysis.md](03-dynamic-analysis.md)*

---

*[Back to README](../README.md)*

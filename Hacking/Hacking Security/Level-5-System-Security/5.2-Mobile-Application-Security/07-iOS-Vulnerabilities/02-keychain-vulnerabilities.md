# Unit 7: iOS Vulnerabilities — Topic 2: Keychain Vulnerabilities

## Overview

While the iOS **Keychain** is the recommended secure storage mechanism, it is not immune to vulnerabilities. Misconfigurations in accessibility classes, overly broad sharing groups, lack of biometric protection, and keychain data persistence after app deletion can all lead to credential exposure.

---

## 1. Common Keychain Misconfigurations

```
KEYCHAIN VULNERABILITY PATTERNS:

1. WRONG ACCESSIBILITY CLASS:
   kSecAttrAccessibleAfterFirstUnlock     ← COMMON MISTAKE
   → Data accessible when device is locked
   → Attacker with physical access can dump
   
   SHOULD BE:
   kSecAttrAccessibleWhenUnlockedThisDeviceOnly
   OR: kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly

2. DATA PERSISTS AFTER UNINSTALL:
   → Keychain items survive app deletion!
   → Re-install app → old tokens still there
   → Attacker installs app → gets previous user's data
   → FIX: Check and clear on first launch
   
   // Detect first launch
   if !UserDefaults.standard.bool(forKey: "notFirstLaunch") {
       // Clear keychain items from previous install
       deleteAllKeychainItems()
       UserDefaults.standard.set(true, forKey: "notFirstLaunch")
   }

3. OVERLY BROAD ACCESS GROUPS:
   kSecAttrAccessGroup: "ABC123.com.example.*"
   → All apps from same team can read!
   → Malicious app from compromised dev account
   → FIX: Use specific access group per app

4. NO BIOMETRIC PROTECTION:
   → Sensitive keys stored without biometric gate
   → Anyone with device passcode can access
   → FIX: Use kSecAccessControlBiometryCurrentSet

5. PLAINTEXT SECRETS:
   → Storing raw passwords in keychain
   → While encrypted at rest, exposed when unlocked
   → FIX: Store derived keys or hashed values

6. MISSING DATA IN KEYCHAIN:
   → Sensitive data stored in NSUserDefaults instead
   → Developer chose convenience over security
   → FIX: Always use Keychain for secrets
```

---

## 2. Testing Keychain Security

```bash
# === KEYCHAIN DUMPING ===

# Using Keychain-Dumper (jailbroken)
# Download from GitHub, copy to device
./keychain_dumper -a
# Output shows ALL accessible keychain items:
# - Service name
# - Account
# - Data (password/token)
# - Access group
# - Protection class

# Using objection
objection -g com.example.app explore
> ios keychain dump
> ios keychain dump --json

# Example output:
# Service: com.example.app.auth
# Account: user@example.com
# Data: "eyJhbGciOiJIUzI1NiIs..."   ← JWT token!
# Accessible: AfterFirstUnlock        ← FINDING!
# Access Group: ABC123.com.example

# Using Frida
frida -U -f com.example.app -l keychain_monitor.js
```

```javascript
// Frida: Monitor all Keychain operations
['SecItemAdd', 'SecItemUpdate', 'SecItemCopyMatching', 'SecItemDelete'].forEach(function(fname) {
    Interceptor.attach(Module.findExportByName('Security', fname), {
        onEnter: function(args) {
            var query = ObjC.Object(args[0]);
            console.log("\n[" + fname + "]");
            console.log("  Class: " + query.objectForKey_("class"));
            console.log("  Service: " + query.objectForKey_("svce"));
            console.log("  Account: " + query.objectForKey_("acct"));
            console.log("  Access Group: " + query.objectForKey_("agrp"));
            console.log("  Accessible: " + query.objectForKey_("pdmn"));
            
            if (fname === 'SecItemAdd') {
                var data = query.objectForKey_("v_Data");
                if (data) {
                    console.log("  Data: " + data.toString());
                }
            }
        },
        onLeave: function(retval) {
            console.log("  Result: " + retval.toInt32());
        }
    });
});
```

---

## 3. Keychain Data Protection Analysis

```
CHECKING PROTECTION LEVELS:

ACCESSIBILITY MAPPING (pdmn values):
  ak    → kSecAttrAccessibleWhenUnlocked
  ck    → kSecAttrAccessibleAfterFirstUnlock
  dk    → kSecAttrAccessibleAlways (DEPRECATED!)
  aku   → kSecAttrAccessibleWhenUnlockedThisDeviceOnly
  cku   → kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly
  dku   → kSecAttrAccessibleAlwaysThisDeviceOnly (DEPRECATED!)
  akpu  → kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly

SEVERITY ASSESSMENT:
┌────────┬──────────────────────┬──────────────────┐
│ pdmn   │ Meaning              │ Severity         │
├────────┼──────────────────────┼──────────────────┤
│ dk/dku │ Always accessible    │ CRITICAL         │
│ ck/cku │ After first unlock   │ HIGH (for creds) │
│ ak     │ When unlocked        │ MEDIUM           │
│ aku    │ Unlocked+device only │ LOW              │
│ akpu   │ Passcode+device only │ INFORMATIONAL    │
└────────┴──────────────────────┴──────────────────┘

WHAT TO REPORT:
  → Auth tokens with ck/cku accessibility
  → Passwords with any non-aku/akpu accessibility
  → Encryption keys without biometric protection
  → Keychain items persisting after uninstall
  → Broad access groups sharing sensitive data
```

---

## 4. Secure Keychain Implementation

```swift
// SECURE: Store with strong protection + biometrics
func storeSecurely(token: String, account: String) {
    let access = SecAccessControlCreateWithFlags(
        nil,
        kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly,
        [.biometryCurrentSet, .privateKeyUsage],
        nil
    )!
    
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrService as String: "com.example.app.auth",
        kSecAttrAccount as String: account,
        kSecValueData as String: token.data(using: .utf8)!,
        kSecAttrAccessControl as String: access,
        kSecAttrAccessGroup as String: "ABC123.com.example.app"
    ]
    
    // Delete existing item first
    SecItemDelete(query as CFDictionary)
    
    let status = SecItemAdd(query as CFDictionary, nil)
    guard status == errSecSuccess else {
        print("Keychain error: \(status)")
        return
    }
}

// SECURE: Clear keychain on first launch after reinstall
func clearKeychainOnReinstall() {
    let freshInstall = !UserDefaults.standard.bool(forKey: "installed")
    if freshInstall {
        let secItemClasses = [
            kSecClassGenericPassword,
            kSecClassInternetPassword,
            kSecClassCertificate,
            kSecClassKey,
            kSecClassIdentity
        ]
        for itemClass in secItemClasses {
            SecItemDelete([kSecClass as String: itemClass] as CFDictionary)
        }
        UserDefaults.standard.set(true, forKey: "installed")
    }
}
```

---

## Summary Table

| Vulnerability | Impact | Detection | Fix |
|--------------|--------|-----------|-----|
| Wrong accessibility | Locked-device access | Keychain dump | Use aku/akpu |
| Post-uninstall persistence | Token reuse | Reinstall test | Clear on first launch |
| Broad access group | Cross-app access | Check agrp | Narrow access group |
| No biometric gate | Passcode-only access | Check flags | Add biometric ACL |
| Plaintext passwords | Direct credential theft | Keychain dump | Hash/derive keys |

---

## Revision Questions

1. What are the most common Keychain misconfigurations?
2. How do Keychain items persist after app uninstallation?
3. What tools dump Keychain contents and what do they reveal?
4. How do you assess the severity of Keychain accessibility settings?
5. What is the most secure Keychain configuration for credentials?

---

*Previous: [01-data-storage-issues.md](01-data-storage-issues.md) | Next: [03-url-schemes.md](03-url-schemes.md)*

---

*[Back to README](../README.md)*

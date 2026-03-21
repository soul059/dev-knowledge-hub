# Unit 5: iOS Security Architecture — Topic 4: Keychain

## Overview

The **iOS Keychain** is a secure, encrypted database for storing sensitive data such as passwords, encryption keys, certificates, and tokens. It's managed by the `securityd` daemon and protected by hardware encryption. Understanding the Keychain is critical for mobile security testing — misconfigured Keychain items are a frequent finding.

---

## 1. Keychain Architecture

```
KEYCHAIN ARCHITECTURE:

┌─────────────────────────────────────────┐
│              APPLICATION                 │
│                                         │
│  SecItemAdd()  SecItemCopyMatching()    │
│  SecItemUpdate()  SecItemDelete()       │
└──────────────────┬──────────────────────┘
                   │ Security Framework API
┌──────────────────▼──────────────────────┐
│           securityd DAEMON              │
│                                         │
│  Access Control     Encryption          │
│  Entitlement Check  Key Management      │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│         KEYCHAIN DATABASE               │
│     /var/Keychains/keychain-2.db        │
│                                         │
│  Encrypted with device-specific keys    │
│  Protected by Data Protection classes   │
└──────────────────┬──────────────────────┘
                   │
┌──────────────────▼──────────────────────┐
│         SECURE ENCLAVE                  │
│  Hardware key storage                   │
│  Biometric-gated access                 │
└─────────────────────────────────────────┘

KEYCHAIN ITEM CLASSES:
  kSecClassGenericPassword    → App passwords, tokens
  kSecClassInternetPassword   → Web credentials
  kSecClassCertificate        → X.509 certificates
  kSecClassKey                → Cryptographic keys
  kSecClassIdentity           → Certificate + private key
```

---

## 2. Keychain Access Control

```
DATA PROTECTION CLASSES FOR KEYCHAIN:

kSecAttrAccessibleWhenUnlocked (Default):
  → Available only when device is unlocked
  → Encrypted when device locks
  → RECOMMENDED for most data

kSecAttrAccessibleAfterFirstUnlock:
  → Available after first unlock until reboot
  → Stays available when locked!
  → Common MISCONFIGURATION for sensitive data

kSecAttrAccessibleAlways (DEPRECATED):
  → Always available, even when locked
  → SECURITY RISK — do not use!

kSecAttrAccessibleWhenUnlockedThisDeviceOnly:
  → Like WhenUnlocked + not included in backups
  → Not transferred to new device
  → BEST for device-bound secrets

kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly:
  → Only available if passcode is set
  → Deleted if passcode is removed
  → MOST SECURE option

ACCESS CONTROL FLAGS:
  .biometryAny          → Any enrolled biometric
  .biometryCurrentSet   → Currently enrolled biometric only
  .devicePasscode       → Device passcode required
  .userPresence         → Biometric OR passcode
  .privateKeyUsage      → For signing operations
```

---

## 3. Testing Keychain Security

```bash
# === ON JAILBROKEN DEVICE ===

# Using Keychain-Dumper
./keychain_dumper -a
# Dumps ALL keychain items accessible to current process
# Look for: passwords, tokens, API keys, certificates

# Using objection
objection -g com.example.app explore
> ios keychain dump
# Shows: service, account, data, accessibility class
# FINDINGS:
# □ Passwords stored in clear
# □ Using kSecAttrAccessibleAlways
# □ Using kSecAttrAccessibleAfterFirstUnlock for sensitive data
# □ Tokens without expiration
# □ Shared keychain groups too broad

# Using Frida
frida -U -f com.example.app -l keychain_dump.js
```

```javascript
// Frida: Monitor Keychain operations
Interceptor.attach(Module.findExportByName("Security", "SecItemAdd"), {
    onEnter: function(args) {
        var query = ObjC.Object(args[0]);
        console.log("[SecItemAdd] Adding keychain item:");
        console.log("  Class: " + query.objectForKey_("class"));
        console.log("  Service: " + query.objectForKey_("svce"));
        console.log("  Account: " + query.objectForKey_("acct"));
        console.log("  Accessible: " + query.objectForKey_("pdmn"));
    }
});

Interceptor.attach(Module.findExportByName("Security", "SecItemCopyMatching"), {
    onEnter: function(args) {
        this.query = ObjC.Object(args[0]);
    },
    onLeave: function(retval) {
        if (retval == 0) {
            console.log("[SecItemCopyMatching] Retrieved keychain item");
            console.log("  Service: " + this.query.objectForKey_("svce"));
        }
    }
});
```

---

## 4. Common Keychain Vulnerabilities

```
VULNERABILITY FINDINGS:

1. WRONG ACCESSIBILITY CLASS:
   → Using AfterFirstUnlock for passwords
   → Data accessible when device is locked
   → FIX: Use WhenUnlockedThisDeviceOnly

2. NO BIOMETRIC PROTECTION:
   → Sensitive keys without biometric gate
   → Anyone with passcode can access
   → FIX: Add .biometryCurrentSet flag

3. KEYCHAIN DATA IN BACKUPS:
   → Items without "ThisDeviceOnly" suffix
   → Included in iTunes/iCloud backups
   → Attacker extracts from backup
   → FIX: Use ThisDeviceOnly variants

4. BROAD KEYCHAIN SHARING:
   → Overly permissive access groups
   → Other apps from same developer can read
   → FIX: Restrict keychain-access-groups

5. PLAINTEXT VALUES:
   → Storing readable passwords in Keychain
   → Better than SharedPreferences but still risky
   → FIX: Store derived keys, not raw passwords
```

---

## Summary Table

| Accessibility Class | When Available | Backup | Security Level |
|-------------------|----------------|--------|---------------|
| WhenUnlocked | Device unlocked | Yes | Good |
| AfterFirstUnlock | After first unlock | Yes | Fair |
| Always (deprecated) | Always | Yes | Poor |
| WhenUnlockedThisDeviceOnly | Device unlocked | No | Very Good |
| WhenPasscodeSetThisDeviceOnly | Passcode set + unlocked | No | Best |

---

## Revision Questions

1. What is the iOS Keychain and what types of data does it store?
2. What are the different Keychain accessibility classes?
3. How do you dump Keychain items on a jailbroken device?
4. What are common Keychain misconfigurations?
5. Why is `ThisDeviceOnly` important for security-sensitive items?

---

*Previous: [03-code-signing.md](03-code-signing.md) | Next: [05-data-protection-api.md](05-data-protection-api.md)*

---

*[Back to README](../README.md)*

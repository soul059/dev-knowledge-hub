# Unit 7: iOS Vulnerabilities — Topic 1: Data Storage Issues

## Overview

**Data storage vulnerabilities** on iOS occur when applications store sensitive information insecurely in the local file system. Despite iOS's strong Data Protection API and Keychain, developers frequently misuse storage mechanisms — saving credentials in plist files, caching sensitive responses, storing data in SQLite without encryption, or using weak Data Protection classes.

---

## 1. iOS Storage Locations and Risks

```
iOS APP DATA STORAGE:

/var/mobile/Containers/Data/Application/<UUID>/
├── Documents/                    ← User documents
│   RISK: Included in iCloud/iTunes backup
│   FINDING: Sensitive files stored here unencrypted
│
├── Library/
│   ├── Preferences/              ← NSUserDefaults (plist)
│   │   RISK: Often stores auth tokens, settings
│   │   FINDING: Plaintext credentials in .plist
│   │
│   ├── Caches/                   ← Cached data
│   │   RISK: HTTP responses cached with sensitive data
│   │   FINDING: API responses with PII in cache
│   │
│   ├── Application Support/      ← App databases
│   │   RISK: SQLite databases without encryption
│   │   FINDING: Unencrypted user data in .db files
│   │
│   ├── Cookies/                  ← Cookie storage
│   │   RISK: Session cookies persisted
│   │   FINDING: Auth cookies without Secure flag
│   │
│   └── WebKit/                   ← WebView data
│       RISK: Cached web content, localStorage
│       FINDING: Sensitive data in web storage
│
├── tmp/                          ← Temporary files
│   RISK: May contain sensitive data between operations
│   FINDING: Decrypted files left in tmp/
│
└── SystemData/                   ← System-managed
    RISK: Snapshots, logs
    FINDING: Screenshots with sensitive data
```

---

## 2. Testing Storage Vulnerabilities

```bash
# === ON JAILBROKEN DEVICE ===

# Find app container
objection -g com.example.app explore
> env   # Shows all paths

# === NSUserDefaults (Plist files) ===
find /var/mobile/Containers/Data/Application/<UUID>/Library/Preferences/ -name "*.plist"
plutil -p com.example.app.plist
# Look for: tokens, passwords, user data, API keys

# === SQLite Databases ===
find /var/mobile/Containers/Data/Application/<UUID>/ -name "*.db" -o -name "*.sqlite"
sqlite3 app.db
> .tables
> .schema users
> SELECT * FROM users;
> SELECT * FROM sessions;
# Look for: plaintext credentials, PII, session data

# === Cache Files ===
ls -la /var/mobile/Containers/Data/Application/<UUID>/Library/Caches/
# Check for cached API responses
find . -name "*.json" -o -name "*.xml" -exec grep -l "password\|token\|email" {} \;

# === WebKit Storage ===
ls -la /var/mobile/Containers/Data/Application/<UUID>/Library/WebKit/
# localStorage, sessionStorage, IndexedDB

# === Snapshot Images ===
ls /var/mobile/Containers/Data/Application/<UUID>/Library/SplashBoard/Snapshots/
# iOS takes screenshots when app backgrounds!
# Sensitive data visible in snapshots

# === Keyboard Cache ===
cat /var/mobile/Library/Keyboard/dynamic-text.dat
# Typed text cached for autocomplete

# === System Logs ===
log show --predicate 'subsystem == "com.example.app"' --last 1h
# Check for sensitive data in logs

# Using objection
> ios nsuserdefaults get
> ios cookies get
> ios plist cat /path/to/file.plist
```

---

## 3. Common Findings

```
TYPICAL VULNERABILITIES:

1. AUTH TOKENS IN NSUSERDEFAULTS:
   <key>auth_token</key>
   <string>eyJhbGciOiJIUzI1NiIs...</string>
   → Should be in Keychain with proper protection

2. UNENCRYPTED SQLITE:
   CREATE TABLE credentials (
     username TEXT, password TEXT, token TEXT
   );
   → Should use SQLCipher or encrypt at app level

3. WRONG DATA PROTECTION CLASS:
   → Default: NSFileProtectionCompleteUntilFirstUserAuthentication
   → Available when device is locked!
   → Sensitive files should use NSFileProtectionComplete

4. BACKGROUND SCREENSHOTS:
   → iOS captures app state when backgrounding
   → Shows sensitive data (banking info, messages)
   → App should use applicationDidEnterBackground to hide

5. CLIPBOARD DATA:
   → Copied passwords remain on clipboard
   → Accessible by other apps (iOS < 14)
   → iOS 14+ shows paste notification
   → App should clear clipboard after paste

6. BACKUP INCLUSION:
   → Sensitive files included in iTunes/iCloud backup
   → Extracted from backup on attacker's computer
   → Set NSURLIsExcludedFromBackupKey = true
```

---

## 4. Secure Storage Recommendations

```swift
// SECURE: Use Keychain for credentials
let query: [String: Any] = [
    kSecClass as String: kSecClassGenericPassword,
    kSecAttrService as String: "com.example.app",
    kSecAttrAccount as String: "authToken",
    kSecValueData as String: tokenData,
    kSecAttrAccessible as String: 
        kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly
]
SecItemAdd(query as CFDictionary, nil)

// SECURE: Hide sensitive UI on background
func applicationDidEnterBackground(_ application: UIApplication) {
    let blurView = UIVisualEffectView(effect: UIBlurEffect(style: .light))
    blurView.frame = window!.frame
    blurView.tag = 999
    window?.addSubview(blurView)
}

// SECURE: Exclude from backup
var resourceValues = URLResourceValues()
resourceValues.isExcludedFromBackup = true
try url.setResourceValues(resourceValues)

// SECURE: Disable keyboard cache on sensitive fields
textField.autocorrectionType = .no
textField.isSecureTextEntry = true

// SECURE: Clear clipboard
UIPasteboard.general.items = []
```

---

## Summary Table

| Storage Location | Default Protection | Risk | Mitigation |
|-----------------|-------------------|------|-----------|
| NSUserDefaults | None | Plaintext in plist | Use Keychain |
| SQLite | File-level only | Unencrypted data | SQLCipher |
| Documents/ | UntilFirstAuth | Backup extraction | Exclude + encrypt |
| Caches/ | None guaranteed | Sensitive cache | Clear on logout |
| Snapshots | None | Visual data leak | Blur on background |
| Clipboard | Shared | Cross-app access | Clear after use |

---

## Revision Questions

1. What are the main data storage locations in an iOS app sandbox?
2. How do you test for insecure data storage on a jailbroken device?
3. Why are background screenshots a security concern?
4. What is the difference between NSUserDefaults and Keychain?
5. How do you prevent sensitive data from appearing in backups?

---

*Previous: None (First topic in this unit) | Next: [02-keychain-vulnerabilities.md](02-keychain-vulnerabilities.md)*

---

*[Back to README](../README.md)*

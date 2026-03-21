# Unit 4: Android Vulnerabilities — Topic 1: Insecure Data Storage

## Overview

**Insecure data storage** is consistently the most prevalent mobile vulnerability. Android apps store data in SQLite databases, SharedPreferences, files, logs, and external storage — often without proper encryption. On a rooted device, all this data is accessible, leading to credential theft, PII exposure, and session hijacking.

---

## 1. Data Storage Locations and Risks

```
ANDROID DATA STORAGE:

/data/data/<package>/shared_prefs/    ← SharedPreferences (XML)
  RISK: Plaintext key-value pairs
  FINDING: Auth tokens, user settings, credentials stored in clear

/data/data/<package>/databases/       ← SQLite databases
  RISK: Unencrypted database files
  FINDING: User data, chat history, financial records in clear

/data/data/<package>/files/           ← Internal files
  RISK: Sensitive files without encryption
  FINDING: Downloaded documents, cached responses

/data/data/<package>/cache/           ← Cache directory
  RISK: Cached API responses with sensitive data
  FINDING: Cached credentials, session data

/sdcard/ (External storage)           ← World-readable!
  RISK: ANY app with READ_EXTERNAL_STORAGE can access
  FINDING: Photos, downloads, exported data in shared space

LogCat                                ← System logs
  RISK: Sensitive data logged in debug builds
  FINDING: API keys, tokens, user data in logs

Clipboard                            ← Shared clipboard
  RISK: Copied passwords accessible to other apps
  FINDING: Passwords, tokens left on clipboard

Keyboard Cache                       ← Predictive text
  RISK: Typed passwords cached by keyboard
  FINDING: Credentials in keyboard dictionary
```

---

## 2. Testing for Insecure Storage

```bash
# === SharedPreferences ===
adb shell cat /data/data/com.example.app/shared_prefs/*.xml
# Look for: auth_token, session_id, password, api_key, user_email

# === SQLite Databases ===
adb shell ls /data/data/com.example.app/databases/
adb pull /data/data/com.example.app/databases/app.db
sqlite3 app.db
sqlite> .tables
sqlite> SELECT * FROM users;
sqlite> SELECT * FROM sessions;
# Look for: plaintext passwords, tokens, PII

# === Files ===
adb shell ls -laR /data/data/com.example.app/files/
adb shell find /data/data/com.example.app/ -type f -name "*.json" -o -name "*.xml" -o -name "*.txt"

# === External Storage ===
adb shell ls -laR /sdcard/Android/data/com.example.app/
adb shell ls /sdcard/ | grep -i "com.example"

# === LogCat ===
adb logcat | grep -i "password\|token\|secret\|key\|bearer"

# === Screenshots ===
# Check if app prevents screenshots on sensitive screens
adb shell screencap -p /sdcard/screenshot.png
# FLAG_SECURE should block this on sensitive activities

# === Backup ===
# Check if backup extracts app data
adb backup -apk -shared com.example.app -f backup.ab
# Convert: java -jar abe.jar unpack backup.ab backup.tar
# Extract and review contents
```

---

## 3. Secure Storage Best Practices

```
SECURE ALTERNATIVES:

FOR CREDENTIALS/TOKENS:
  → Android Keystore System (hardware-backed)
  → EncryptedSharedPreferences (Jetpack Security)
  → Never store plaintext passwords

FOR DATABASES:
  → SQLCipher (encrypted SQLite)
  → Realm with encryption
  → Room with encryption layer

FOR FILES:
  → EncryptedFile (Jetpack Security)
  → AES-256 encryption with Keystore-managed keys

ANDROID JETPACK SECURITY:
  // EncryptedSharedPreferences
  val masterKey = MasterKey.Builder(context)
      .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
      .build()
  
  val sharedPrefs = EncryptedSharedPreferences.create(
      context, "secure_prefs", masterKey,
      EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
      EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
  )
```

---

## Summary Table

| Storage | Default Security | Risk | Mitigation |
|---------|-----------------|------|-----------|
| SharedPreferences | None | Plaintext exposure | EncryptedSharedPreferences |
| SQLite | None | Full DB accessible | SQLCipher |
| Internal Files | App-only access | Root access reveals | EncryptedFile |
| External Storage | World-readable | Any app can read | Don't store sensitive data |
| Logs | Readable on debug | Info leakage | Remove debug logging |
| Clipboard | Shared | Cross-app access | Clear after use |

---

## Revision Questions

1. What are the main data storage locations on Android?
2. How do you test for insecure SharedPreferences storage?
3. Why is external storage dangerous for sensitive data?
4. What is EncryptedSharedPreferences and how does it work?
5. How do you check for sensitive data in LogCat?

---

*Previous: None (First topic in this unit) | Next: [02-insecure-communication.md](02-insecure-communication.md)*

---

*[Back to README](../README.md)*

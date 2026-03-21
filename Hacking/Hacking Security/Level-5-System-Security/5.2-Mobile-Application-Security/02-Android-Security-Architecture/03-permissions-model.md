# Unit 2: Android Security Architecture — Topic 3: Permissions Model

## Overview

Android's **permission model** controls what resources and data an app can access. Starting with Android 6.0 (Marshmallow), dangerous permissions require **runtime approval** from the user, rather than being granted at install time. Understanding the permission model is critical for identifying over-permissioned apps and testing permission enforcement.

---

## 1. Permission Categories

```
ANDROID PERMISSION LEVELS:

NORMAL PERMISSIONS (Auto-granted, no user prompt):
├── INTERNET — Network access
├── ACCESS_NETWORK_STATE — Check network status
├── BLUETOOTH — Bluetooth connections
├── NFC — Near field communication
├── SET_ALARM — Set alarms
├── VIBRATE — Vibrate device
├── WAKE_LOCK — Prevent sleep
└── RECEIVE_BOOT_COMPLETED — Run at boot

DANGEROUS PERMISSIONS (Require user approval):
├── CAMERA — Camera access
├── READ_CONTACTS — Read contacts
├── ACCESS_FINE_LOCATION — GPS location
├── READ_SMS — Read text messages
├── RECORD_AUDIO — Microphone access
├── READ_EXTERNAL_STORAGE — Read files
├── WRITE_EXTERNAL_STORAGE — Write files
├── READ_CALL_LOG — Call history
├── READ_PHONE_STATE — Phone info (IMEI, etc.)
└── BODY_SENSORS — Health sensors

SIGNATURE PERMISSIONS (Only same-key apps):
├── Granted only to apps signed with same certificate
├── Used for inter-app communication
└── System apps use for system-level access

SPECIAL PERMISSIONS (Require settings toggle):
├── SYSTEM_ALERT_WINDOW — Draw over other apps
├── WRITE_SETTINGS — Modify system settings
├── REQUEST_INSTALL_PACKAGES — Install APKs
├── MANAGE_EXTERNAL_STORAGE — Full file access
└── ACCESS_NOTIFICATION_LISTENER — Read notifications
```

---

## 2. Permission Groups

```
DANGEROUS PERMISSIONS ARE GROUPED:

LOCATION:
├── ACCESS_FINE_LOCATION
├── ACCESS_COARSE_LOCATION
└── ACCESS_BACKGROUND_LOCATION

CAMERA:
└── CAMERA

MICROPHONE:
└── RECORD_AUDIO

CONTACTS:
├── READ_CONTACTS
├── WRITE_CONTACTS
└── GET_ACCOUNTS

PHONE:
├── READ_PHONE_STATE
├── CALL_PHONE
├── READ_CALL_LOG
└── WRITE_CALL_LOG

SMS:
├── SEND_SMS
├── RECEIVE_SMS
├── READ_SMS
└── RECEIVE_MMS

STORAGE:
├── READ_EXTERNAL_STORAGE
├── WRITE_EXTERNAL_STORAGE
└── READ_MEDIA_IMAGES/VIDEO/AUDIO (Android 13+)

RULE: Granting one permission in a group historically
granted all permissions in that group. Android 12+
changed this — each permission requires individual grant.
```

---

## 3. Security Testing Permissions

```bash
# List all permissions an app requests
adb shell dumpsys package com.example.app | grep "permission"

# Check granted vs requested permissions
adb shell dumpsys package com.example.app | grep -A 50 "requested permissions"
adb shell dumpsys package com.example.app | grep -A 50 "install permissions"
adb shell dumpsys package com.example.app | grep -A 50 "runtime permissions"

# Grant a permission via ADB
adb shell pm grant com.example.app android.permission.READ_CONTACTS

# Revoke a permission via ADB
adb shell pm revoke com.example.app android.permission.READ_CONTACTS

# List all permissions on device
adb shell pm list permissions -d -g  # Dangerous permissions grouped

# Check AndroidManifest.xml for declared permissions
# After decompiling with apktool:
# Look for <uses-permission> tags
```

```
SECURITY TESTING CHECKLIST:
□ Does the app request unnecessary permissions?
□ Does the app function with permissions denied?
□ Does the app handle permission denial gracefully?
□ Are dangerous permissions justified by functionality?
□ Does the app use runtime permissions (not install-time)?
□ Are custom permissions properly defined?
□ Do exported components enforce permissions?
```

---

## 4. Common Permission Vulnerabilities

```
OVER-PERMISSIONING:
  → App requests permissions it doesn't need
  → Example: Calculator requesting CAMERA and CONTACTS
  → Increases risk if app is compromised
  → Violates principle of least privilege

MISSING PERMISSION ENFORCEMENT:
  → Exported component doesn't check permissions
  → Any app can call the Activity/Service
  → Data leakage through unprotected components

  <!-- VULNERABLE: No permission required -->
  <activity android:name=".AdminActivity"
            android:exported="true" />

  <!-- SECURE: Permission required -->
  <activity android:name=".AdminActivity"
            android:exported="true"
            android:permission="com.example.ADMIN" />

CUSTOM PERMISSION ISSUES:
  → Custom permissions with wrong protection level
  → Using "normal" instead of "signature" for sensitive ops
  → Permission name collision between apps
```

---

## Summary Table

| Level | User Prompt | Example | Risk |
|-------|------------|---------|------|
| Normal | No | INTERNET | Low |
| Dangerous | Yes (runtime) | CAMERA, LOCATION | Medium-High |
| Signature | No (same cert) | System features | Low |
| Special | Settings toggle | SYSTEM_ALERT_WINDOW | High |

---

## Revision Questions

1. What are the four permission protection levels in Android?
2. How did Android 6.0 change the permission model?
3. What is permission over-privileging and how do you detect it?
4. How do you test permission enforcement on exported components?
5. What are the security implications of permission groups?

---

*Previous: [02-application-sandbox.md](02-application-sandbox.md) | Next: [04-android-security-features.md](04-android-security-features.md)*

---

*[Back to README](../README.md)*

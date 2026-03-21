# Unit 3: Android Testing — Topic 3: Dynamic Analysis

## Overview

**Dynamic analysis** tests an Android application **while it's running** on a device or emulator. This reveals runtime behavior including network communications, file system changes, inter-process communication, cryptographic operations, and memory contents that static analysis cannot uncover.

---

## 1. Dynamic Analysis Areas

```
DYNAMIC ANALYSIS SCOPE:

RUNTIME BEHAVIOR:
├── Network traffic (API calls, data transmitted)
├── File system changes (data written/read)
├── Database operations (SQLite queries)
├── SharedPreferences changes
├── Log output (LogCat)
├── Clipboard access
├── IPC communications (Intents)
└── Cryptographic operations

RUNTIME MANIPULATION:
├── Hook function calls (Frida)
├── Modify return values
├── Bypass security checks
├── Inject code at runtime
├── Dump encryption keys
└── Intercept API responses
```

---

## 2. Monitoring with ADB

```bash
# === LOGCAT — Application Logs ===
adb logcat | grep com.example.app
adb logcat --pid=$(adb shell pidof com.example.app)
# Look for: passwords, tokens, sensitive data in logs

# === FILE SYSTEM MONITORING ===
# Check app data before and after actions
adb shell ls -laR /data/data/com.example.app/
adb shell cat /data/data/com.example.app/shared_prefs/*.xml
adb shell sqlite3 /data/data/com.example.app/databases/app.db ".dump"

# Take snapshot before/after login:
adb shell find /data/data/com.example.app/ -newer /tmp/timestamp -ls

# === CLIPBOARD ===
adb shell service call clipboard 2 s16 com.example.app

# === INTENT MONITORING ===
# Watch Intents being broadcast
adb shell dumpsys activity broadcasts

# Send Intents to test exported components
adb shell am start -n com.example.app/.AdminActivity
adb shell am startservice -n com.example.app/.BackgroundService
adb shell am broadcast -a com.example.CUSTOM_ACTION

# === CONTENT PROVIDER TESTING ===
adb shell content query --uri content://com.example.app.provider/users
adb shell content query --uri content://com.example.app.provider/users --where "_id=1"
```

---

## 3. Frida Dynamic Instrumentation

```javascript
// === FRIDA BASICS ===
// Attach to running process
// frida -U -f com.example.app -l script.js --no-pause

// Hook a method and log arguments
Java.perform(function() {
    var LoginActivity = Java.use("com.example.app.LoginActivity");
    
    LoginActivity.login.implementation = function(username, password) {
        console.log("[*] Login called!");
        console.log("[*] Username: " + username);
        console.log("[*] Password: " + password);
        
        // Call original method
        return this.login(username, password);
    };
});

// === BYPASS SSL PINNING (generic) ===
Java.perform(function() {
    var TrustManager = Java.use('javax.net.ssl.X509TrustManager');
    var SSLContext = Java.use('javax.net.ssl.SSLContext');
    
    // Create custom TrustManager that trusts all
    var TrustAllManager = Java.registerClass({
        name: 'com.custom.TrustAllManager',
        implements: [TrustManager],
        methods: {
            checkClientTrusted: function(chain, authType) {},
            checkServerTrusted: function(chain, authType) {},
            getAcceptedIssuers: function() { return []; }
        }
    });
    
    console.log("[*] SSL Pinning Bypassed");
});

// === DUMP SHARED PREFERENCES ===
Java.perform(function() {
    var SharedPreferences = Java.use('android.app.SharedPreferencesImpl');
    SharedPreferences.getString.implementation = function(key, defValue) {
        var result = this.getString(key, defValue);
        console.log("[SharedPrefs] " + key + " = " + result);
        return result;
    };
});
```

---

## 4. Objection Runtime Exploration

```bash
# Start Objection
objection -g com.example.app explore

# === COMMON COMMANDS ===
# File system
objection> env                           # Show app directories
objection> ls /data/data/com.example.app/
objection> file download /data/data/com.example.app/databases/app.db

# SQLite
objection> sqlite connect app.db
objection> sqlite execute "SELECT * FROM users"

# SharedPreferences
objection> android sslpinning disable    # Bypass SSL pinning
objection> android root disable          # Bypass root detection

# Keystore
objection> android keystore list
objection> android keystore dump

# Activities
objection> android hooking list activities
objection> android intent launch_activity com.example.app.HiddenActivity

# Hooking
objection> android hooking watch class com.example.app.Auth
objection> android hooking watch class_method com.example.app.Auth.login --dump-args --dump-return
```

---

## Summary Table

| Technique | Tool | What It Reveals |
|-----------|------|----------------|
| Log monitoring | adb logcat | Sensitive data in logs |
| File system | adb shell | Stored data, databases |
| Intent testing | adb am | Exported component access |
| Function hooking | Frida | Runtime behavior, secrets |
| Runtime exploration | Objection | Comprehensive app analysis |
| Content providers | adb content | Data exposure |

---

## Revision Questions

1. What areas does dynamic analysis cover that static analysis cannot?
2. How do you monitor an app's LogCat output for sensitive data?
3. Write a Frida script to hook a login function and log credentials.
4. How do you test exported Android components using ADB?
5. What Objection commands are most useful for security testing?

---

*Previous: [02-static-analysis.md](02-static-analysis.md) | Next: [04-traffic-interception.md](04-traffic-interception.md)*

---

*[Back to README](../README.md)*

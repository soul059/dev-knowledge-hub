# Unit 4: Android Vulnerabilities — Topic 6: Intent Vulnerabilities

## Overview

**Intents** are Android's inter-process communication (IPC) mechanism used to launch activities, start services, and send broadcasts. When app components are **exported** without proper permission checks, or when Intent data is trusted without validation, attackers can exploit these to access sensitive functionality, steal data, or inject malicious content.

---

## 1. Intent Types and Security

```
INTENT TYPES:

EXPLICIT INTENT:
  → Targets a specific component by class name
  → Intent intent = new Intent(this, TargetActivity.class);
  → Generally safe (internal communication)

IMPLICIT INTENT:
  → Specifies action, not target component
  → Intent intent = new Intent("com.example.ACTION_VIEW");
  → Android finds matching component
  → Can be intercepted by malicious apps!

BROADCAST INTENT:
  → Sent to all registered receivers
  → sendBroadcast(new Intent("com.example.DATA_READY"));
  → Any app can register to receive!
  → Sensitive data in broadcasts = leakage!

PENDING INTENT:
  → Wraps an Intent for later execution
  → Can be dangerous if mutable and exported
  → Other apps can modify and execute
```

---

## 2. Common Intent Vulnerabilities

```
1. EXPORTED ACTIVITY WITHOUT PROTECTION:
   <!-- VULNERABLE -->
   <activity android:name=".AdminActivity"
             android:exported="true" />
   
   EXPLOIT:
   adb shell am start -n com.example.app/.AdminActivity
   → Any app/user can access admin functionality!

2. INTENT DATA INJECTION:
   <!-- App processes Intent extras without validation -->
   String url = getIntent().getStringExtra("url");
   webView.loadUrl(url);  // Loads attacker-controlled URL!
   
   EXPLOIT:
   adb shell am start -n com.example.app/.WebViewActivity \
       --es url "javascript:alert(document.cookie)"

3. BROADCAST DATA LEAKAGE:
   // VULNERABLE: Sensitive data in broadcast
   Intent intent = new Intent("com.example.USER_DATA");
   intent.putExtra("token", authToken);
   sendBroadcast(intent);  // ANY app can receive this!
   
   FIX: Use LocalBroadcastManager or explicit permissions:
   sendBroadcast(intent, "com.example.PERMISSION");

4. CONTENT PROVIDER INJECTION:
   <!-- VULNERABLE: Exported without permission -->
   <provider android:name=".UserProvider"
             android:authorities="com.example.provider"
             android:exported="true" />
   
   EXPLOIT:
   adb shell content query --uri content://com.example.provider/users
   → Read all user data!
   
   SQL INJECTION:
   content query --uri content://com.example.provider/users \
       --where "1=1) UNION SELECT password FROM credentials--"

5. PENDING INTENT HIJACK:
   // VULNERABLE: Mutable empty PendingIntent
   PendingIntent pi = PendingIntent.getActivity(this, 0,
       new Intent(), PendingIntent.FLAG_MUTABLE);
   // Malicious app can fill in the empty Intent!
```

---

## 3. Testing Intent Vulnerabilities

```bash
# === DROZER (Android Security Testing Framework) ===

# Find exported components
dz> run app.package.attacksurface com.example.app
# Shows: exported Activities, Services, Receivers, Providers

# Launch exported activities
dz> run app.activity.start --component com.example.app .AdminActivity

# Test content providers
dz> run app.provider.query content://com.example.app.provider/users
dz> run app.provider.finduri com.example.app
# Test for SQL injection:
dz> run scanner.provider.injection -a com.example.app

# === ADB TESTING ===

# List exported components from manifest
grep -A5 "exported=\"true\"" AndroidManifest.xml

# Start activities with data
adb shell am start -n com.example.app/.DeepLinkActivity \
    -d "example://action?param=injected_value"

# Send broadcasts
adb shell am broadcast -a com.example.CUSTOM_ACTION \
    --es key "malicious_value"

# Query content providers
adb shell content query --uri content://com.example.app.provider/
```

---

## 4. Secure Intent Handling

```java
// SECURE: Validate Intent data
String url = getIntent().getStringExtra("url");
if (url != null && url.startsWith("https://trusted.example.com")) {
    webView.loadUrl(url);
} else {
    // Reject untrusted input
    finish();
}

// SECURE: Use explicit broadcasts
Intent intent = new Intent(this, MyReceiver.class);
intent.putExtra("data", sensitiveData);
sendBroadcast(intent);  // Only MyReceiver receives

// SECURE: Protect exported components
<activity android:name=".AdminActivity"
          android:exported="true"
          android:permission="com.example.ADMIN_PERMISSION" />

// SECURE: Protect content providers
<provider android:name=".UserProvider"
          android:exported="false" />
<!-- Or with read/write permissions -->
<provider android:name=".UserProvider"
          android:exported="true"
          android:readPermission="com.example.READ"
          android:writePermission="com.example.WRITE" />

// SECURE: Immutable PendingIntents
PendingIntent pi = PendingIntent.getActivity(this, 0,
    explicitIntent, PendingIntent.FLAG_IMMUTABLE);
```

---

## Summary Table

| Vulnerability | Attack | Impact | Fix |
|--------------|--------|--------|-----|
| Exported Activity | Launch directly | Unauthorized access | Add permission check |
| Intent injection | Send crafted data | Code execution, data theft | Validate input |
| Broadcast leakage | Register receiver | Data interception | Use LocalBroadcast |
| Content Provider | Query/inject | Data theft, SQLi | Set exported=false |
| PendingIntent | Modify intent | Privilege escalation | Use FLAG_IMMUTABLE |

---

## Revision Questions

1. What is the difference between explicit and implicit Intents?
2. How do you find and test exported components?
3. What is a broadcast data leakage vulnerability?
4. How can content providers be exploited via SQL injection?
5. What makes mutable PendingIntents dangerous?
6. What tools are used to test Intent vulnerabilities?

---

*Previous: [05-reverse-engineering.md](05-reverse-engineering.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

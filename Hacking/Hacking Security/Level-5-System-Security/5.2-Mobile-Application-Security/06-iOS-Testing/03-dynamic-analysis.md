# Unit 6: iOS Testing — Topic 3: Dynamic Analysis

## Overview

**Dynamic analysis** of iOS applications involves testing the running application to observe behavior, intercept function calls, modify runtime data, and identify vulnerabilities that aren't visible through static analysis alone. This requires a jailbroken device with tools like Frida, objection, and Cycript.

---

## 1. Dynamic Analysis Workflow

```
iOS DYNAMIC ANALYSIS FLOW:

1. PREPARE ENVIRONMENT
   ├── Jailbroken device with SSH
   ├── Frida server running on device
   ├── Proxy configured (Burp Suite)
   └── Target app installed

2. RUNTIME EXPLORATION
   ├── List loaded classes and methods
   ├── Explore UI hierarchy
   ├── Identify interesting methods
   └── Map authentication flow

3. METHOD HOOKING
   ├── Hook authentication methods
   ├── Hook encryption/decryption
   ├── Hook jailbreak detection
   ├── Monitor data storage calls
   └── Track network requests

4. DATA MANIPULATION
   ├── Modify function return values
   ├── Change method arguments
   ├── Bypass security controls
   └── Inject test data

5. DOCUMENTATION
   └── Record findings, PoCs
```

---

## 2. Frida on iOS

```bash
# === FRIDA SETUP ===

# Install Frida on device (via Cydia/Sileo)
# Add repo: https://build.frida.re
# Install: Frida package

# Install Frida tools on macOS
pip3 install frida-tools

# Verify connection
frida-ls-devices
frida-ps -U                  # List processes on USB device
frida-ps -Ua                 # List running apps
frida-ps -Uai                # List all installed apps

# === ATTACH TO APP ===
frida -U -f com.example.app   # Spawn and attach
frida -U -n "AppName"          # Attach to running app

# === BASIC EXPLORATION ===
```

```javascript
// List all classes
Java.perform(function() {});  // Not needed on iOS

// List ObjC classes containing "Login"
ObjC.enumerateLoadedClasses({
    onMatch: function(className) {
        if (className.includes("Login") || className.includes("Auth")) {
            console.log("[Class] " + className);
        }
    },
    onComplete: function() {}
});

// List methods of a class
var methods = ObjC.classes.LoginViewController.$ownMethods;
methods.forEach(function(method) {
    console.log("[Method] " + method);
});

// Hook a method
Interceptor.attach(
    ObjC.classes.LoginViewController['- validatePassword:'].implementation, {
    onEnter: function(args) {
        // args[0] = self, args[1] = selector, args[2] = first param
        var password = ObjC.Object(args[2]);
        console.log("[*] Password: " + password.toString());
    },
    onLeave: function(retval) {
        console.log("[*] Return: " + retval);
        // Force return true
        retval.replace(0x1);
    }
});

// Monitor Keychain access
Interceptor.attach(Module.findExportByName("Security", "SecItemCopyMatching"), {
    onEnter: function(args) {
        var query = ObjC.Object(args[0]);
        console.log("[Keychain Read] " + query.toString());
    },
    onLeave: function(retval) {
        if (retval.toInt32() === 0) { // errSecSuccess
            console.log("[Keychain] Item found!");
        }
    }
});
```

---

## 3. Objection Framework

```bash
# objection — Automated iOS testing powered by Frida

# Install
pip3 install objection

# Connect to app
objection -g com.example.app explore

# === COMMON COMMANDS ===

# Environment
> env                           # App paths and environment

# Keychain
> ios keychain dump             # Dump all keychain items
> ios keychain dump --json      # JSON output

# Cookies
> ios cookies get               # Get app cookies

# Plist files
> ios plist cat <path>          # Read plist file

# NSUserDefaults
> ios nsuserdefaults get        # Dump UserDefaults

# UI analysis
> ios ui dump                   # Dump view hierarchy
> ios ui alert "Test Message"   # Show alert on device

# Jailbreak detection
> ios jailbreak disable         # Attempt bypass
> ios jailbreak simulate        # Test detection

# SSL Pinning
> ios sslpinning disable        # Disable certificate pinning

# Pasteboard
> ios pasteboard monitor        # Monitor clipboard

# Bundles
> ios bundles list_frameworks   # List loaded frameworks

# Hooking
> ios hooking list classes      # List all classes
> ios hooking search classes Login  # Search classes
> ios hooking list class_methods LoginViewController
> ios hooking watch method "-[LoginViewController validatePassword:]" --dump-args --dump-return
```

---

## 4. Runtime Manipulation Techniques

```
COMMON DYNAMIC ANALYSIS TASKS:

BYPASS LOGIN:
  → Hook validateCredentials → return true
  → Hook isAuthenticated → return true
  → Modify token validation → accept any token

BYPASS JAILBREAK DETECTION:
  → Hook detection methods → return false
  → Hook file existence checks → deny jailbreak files
  → Use Liberty Lite / A-Bypass tweaks
  → objection: ios jailbreak disable

BYPASS ROOT/ADMIN CHECKS:
  → Hook isAdmin/isPremium → return true
  → Modify user role at runtime
  → Change server response

DUMP ENCRYPTION KEYS:
  → Hook CCCrypt → log key and IV
  → Hook SecKeyEncrypt → log plaintext
  → Hook AES init → extract key material

MONITOR FILE OPERATIONS:
  → Hook NSFileManager methods
  → Track what files are read/written
  → Check protection classes used

INTERCEPT BIOMETRICS:
  → Hook LAContext evaluatePolicy
  → Force success callback
  → Bypass Touch ID / Face ID gate
```

```javascript
// Frida: Bypass biometric authentication
var LAContext = ObjC.classes.LAContext;
Interceptor.attach(LAContext['- evaluatePolicy:localizedReason:reply:'].implementation, {
    onEnter: function(args) {
        var callback = new ObjC.Block(args[4]);
        var origCallback = callback.implementation;
        callback.implementation = function(success, error) {
            console.log("[*] Biometric bypassed!");
            origCallback(true, null);  // Always succeed
        };
    }
});
```

---

## Summary Table

| Tool | Primary Use | Requires Jailbreak? |
|------|------------|-------------------|
| Frida | Runtime hooking, exploration | Yes |
| objection | Automated testing | Yes |
| Cycript | Interactive exploration | Yes (older) |
| lldb | Debugging | Development entitlement |
| dtrace | System call tracing | Yes |

---

## Revision Questions

1. What is the iOS dynamic analysis workflow?
2. How do you use Frida to hook Objective-C methods?
3. What objection commands are most useful for iOS testing?
4. How do you bypass biometric authentication with Frida?
5. What runtime data can be extracted through dynamic analysis?

---

*Previous: [02-static-analysis.md](02-static-analysis.md) | Next: [04-traffic-interception.md](04-traffic-interception.md)*

---

*[Back to README](../README.md)*

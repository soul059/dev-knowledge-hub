# Unit 7: iOS Vulnerabilities — Topic 4: Runtime Manipulation

## Overview

**Runtime manipulation** attacks exploit the Objective-C and Swift runtime to modify app behavior, bypass security controls, extract sensitive data, and forge function results — all without modifying the app binary. This is one of the most powerful attack vectors on iOS and is extensively used during penetration testing.

---

## 1. Objective-C Runtime Exploitation

```
WHY ObjC RUNTIME IS EXPLOITABLE:

OBJECTIVE-C FEATURES:
  → Dynamic message dispatch (objc_msgSend)
  → Method swizzling (replace method implementations)
  → Runtime reflection (inspect classes at runtime)
  → Dynamic class creation and modification
  → Late binding (methods resolved at runtime)

ATTACK SURFACE:
  → Hook any ObjC method
  → Replace method implementations
  → Modify instance variables
  → Intercept message passing
  → Create fake objects

SWIFT vs OBJECTIVE-C:
  → Pure Swift: Static dispatch (harder to hook)
  → Swift with @objc: Uses ObjC runtime (hookable)
  → Most iOS apps use ObjC or mixed ObjC/Swift
  → UIKit is ObjC-based → UI methods always hookable
```

---

## 2. Method Swizzling Attacks

```
METHOD SWIZZLING CONCEPT:

BEFORE SWIZZLING:
  [LoginVC validatePassword:] → Original implementation
  
AFTER SWIZZLING:
  [LoginVC validatePassword:] → Attacker's implementation
  [LoginVC original_validatePassword:] → Original (saved)

ATTACKER'S IMPLEMENTATION:
  1. Log arguments (capture password)
  2. Call original method
  3. Modify return value (bypass auth)
  4. Return to caller
```

```javascript
// Frida: Method swizzling examples

// === BYPASS AUTHENTICATION ===
Interceptor.attach(
    ObjC.classes.AuthenticationManager['- isUserAuthenticated'].implementation, {
    onLeave: function(retval) {
        console.log("[*] Auth check bypassed (was: " + retval + ")");
        retval.replace(0x1);  // Always authenticated
    }
});

// === CAPTURE CREDENTIALS ===
Interceptor.attach(
    ObjC.classes.LoginViewController['- loginWithUsername:password:'].implementation, {
    onEnter: function(args) {
        var username = ObjC.Object(args[2]).toString();
        var password = ObjC.Object(args[3]).toString();
        console.log("[CAPTURED] " + username + ":" + password);
    }
});

// === BYPASS PREMIUM CHECK ===
Interceptor.attach(
    ObjC.classes.SubscriptionManager['- isPremiumUser'].implementation, {
    onLeave: function(retval) {
        retval.replace(0x1);  // Premium unlocked
    }
});

// === MODIFY FINANCIAL TRANSACTIONS ===
Interceptor.attach(
    ObjC.classes.PaymentController['- processPayment:amount:'].implementation, {
    onEnter: function(args) {
        var originalAmount = ObjC.Object(args[3]);
        console.log("[*] Original amount: " + originalAmount);
        // Could modify amount here
    }
});

// === BYPASS ROOT/JAILBREAK DETECTION ===
var methods = ObjC.classes.SecurityChecker.$ownMethods;
methods.forEach(function(method) {
    if (method.includes("jailbreak") || method.includes("root") || 
        method.includes("detect") || method.includes("security")) {
        console.log("[*] Hooking: " + method);
        Interceptor.attach(
            ObjC.classes.SecurityChecker[method].implementation, {
            onLeave: function(retval) {
                retval.replace(0x0);  // No jailbreak detected
            }
        });
    }
});
```

---

## 3. Instance Variable Manipulation

```javascript
// Frida: Read and modify instance variables (ivars)

// List all ivars of an object
function dumpIvars(obj) {
    var o = ObjC.Object(obj);
    var ivars = o.$ivars;
    console.log("Instance variables for " + o.$className + ":");
    for (var key in ivars) {
        console.log("  " + key + " = " + ivars[key]);
    }
}

// Modify ivar value
Interceptor.attach(
    ObjC.classes.UserSession['- checkPermission:'].implementation, {
    onEnter: function(args) {
        var self = ObjC.Object(args[0]);
        
        // Read current role
        var role = self.$ivars['_userRole'];
        console.log("[*] Current role: " + role);
        
        // Modify to admin
        self.$ivars['_userRole'] = ObjC.classes.NSString.stringWithString_("admin");
        self.$ivars['_isAdmin'] = true;
        console.log("[*] Role changed to admin!");
    }
});

// Find and modify singleton instances
var sharedSession = ObjC.classes.UserSession.sharedInstance();
console.log("Current user: " + sharedSession.$ivars['_username']);
console.log("Is premium: " + sharedSession.$ivars['_isPremium']);
sharedSession.$ivars['_isPremium'] = true;
```

---

## 4. Protections Against Runtime Manipulation

```
DEFENSE MECHANISMS:

1. INTEGRITY CHECKS:
   → Verify method implementations haven't changed
   → Compare function pointers against known-good values
   → Detect Frida/Cycript presence

2. ANTI-DEBUGGING:
   → ptrace(PT_DENY_ATTACH, 0, 0, 0)
   → sysctl to check for debugger
   → Monitor for debug exceptions
   
   BYPASS: Hook ptrace → ignore deny_attach

3. OBFUSCATION:
   → Rename methods to meaningless names
   → Move logic to C/C++ (harder to hook)
   → Use Swift with static dispatch

4. SERVER-SIDE VALIDATION:
   → Never trust client-side checks alone
   → Validate all security decisions on server
   → Server verifies transactions, permissions
   → Most effective defense!

5. DETECTION OF HOOKING:
   → Check for Frida by scanning for frida-agent
   → Detect Substrate/Substitute libraries
   → Verify code pages in memory
   
   BYPASS: Usually hookable too (meta-bypass)
```

---

## Summary Table

| Attack | Technique | Tool | Impact |
|--------|-----------|------|--------|
| Auth bypass | Hook login method | Frida | Full access |
| Credential theft | Log method args | Frida | Account compromise |
| Premium bypass | Modify return value | Frida | Financial loss |
| Role escalation | Modify ivars | Frida | Privilege escalation |
| Logic bypass | Swizzle method | Frida/Cycript | Business logic abuse |

---

## Revision Questions

1. Why is the Objective-C runtime particularly vulnerable to manipulation?
2. How does method swizzling work and how is it exploited?
3. How do you modify instance variables at runtime with Frida?
4. What protections can apps implement against runtime manipulation?
5. Why is server-side validation the most effective defense?

---

*Previous: [03-url-schemes.md](03-url-schemes.md) | Next: [05-binary-analysis.md](05-binary-analysis.md)*

---

*[Back to README](../README.md)*

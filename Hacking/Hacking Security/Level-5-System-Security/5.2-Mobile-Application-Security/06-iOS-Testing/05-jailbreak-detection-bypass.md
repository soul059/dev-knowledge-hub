# Unit 6: iOS Testing — Topic 5: Jailbreak Detection Bypass

## Overview

Many iOS apps implement **jailbreak detection** to prevent running on compromised devices. While this is a valid defense measure, security testers need to bypass these checks to perform thorough assessments. Understanding both the detection methods and bypass techniques is essential for mobile penetration testing.

---

## 1. Jailbreak Detection Methods

```
DETECTION TECHNIQUES:

1. FILE-BASED CHECKS:
   → Check for existence of jailbreak files:
     /Applications/Cydia.app
     /Library/MobileSubstrate/
     /bin/bash
     /usr/sbin/sshd
     /etc/apt
     /private/var/lib/apt/
     /usr/bin/ssh
     /var/cache/apt
     /var/lib/cydia
     /var/tmp/cydia.log
     /.installed_zydia
     /private/var/stash

2. URL SCHEME CHECKS:
   → Try opening Cydia URL scheme:
     UIApplication.shared.canOpenURL(URL(string: "cydia://")!)
   → Also check: sileo://, zbra://, filza://

3. SANDBOX INTEGRITY:
   → Try writing to paths outside sandbox:
     /private/jailbreak_test.txt
   → If successful → sandbox is broken → jailbroken

4. DYLIB INJECTION DETECTION:
   → Check loaded dynamic libraries:
     _dyld_get_image_name() for each loaded image
   → Look for: MobileSubstrate, Frida, Cycript

5. SYSTEM CALL CHECKS:
   → fork() succeeds on jailbroken devices
   → system() available on jailbroken devices
   → stat() on jailbreak paths

6. ENVIRONMENT CHECKS:
   → DYLD_INSERT_LIBRARIES environment variable
   → Indicates library injection

7. SYMBOLIC LINK CHECKS:
   → /Applications, /var/stash are symlinks on jailbroken
   → lstat() vs stat() comparison
```

---

## 2. Bypass Techniques

```bash
# === TWEAK-BASED BYPASS ===

# Liberty Lite (Cydia/Sileo)
# → Hooks file system calls for selected apps
# → Per-app toggle in Settings

# A-Bypass
# → Advanced detection bypass
# → Hooks multiple detection methods

# Shadow
# → Comprehensive bypass tweak
# → Hooks file, URL, dylib checks

# FlyJB X
# → Another popular bypass tweak
# → Good compatibility

# === OBJECTION BYPASS ===
objection -g com.example.app explore
> ios jailbreak disable
# Hooks common detection methods:
# → NSFileManager fileExistsAtPath
# → UIApplication canOpenURL
# → fork() system call
```

```javascript
// Frida: Comprehensive jailbreak detection bypass

Java.perform(function() {});  // not needed on iOS

// Hook NSFileManager to deny jailbreak files
var fileManager = ObjC.classes.NSFileManager;
Interceptor.attach(fileManager['- fileExistsAtPath:'].implementation, {
    onEnter: function(args) {
        this.path = ObjC.Object(args[2]).toString();
    },
    onLeave: function(retval) {
        var dominated = [
            "/Applications/Cydia.app",
            "/Library/MobileSubstrate",
            "/bin/bash", "/usr/sbin/sshd",
            "/etc/apt", "/usr/bin/ssh",
            "/var/cache/apt", "/var/lib/cydia",
            "/private/var/lib/apt"
        ];
        for (var i = 0; i < dominated.length; i++) {
            if (this.path.indexOf(dominated[i]) !== -1) {
                retval.replace(0x0);  // File doesn't exist
                console.log("[JB Bypass] Denied: " + this.path);
                return;
            }
        }
    }
});

// Hook canOpenURL to deny Cydia
Interceptor.attach(ObjC.classes.UIApplication['- canOpenURL:'].implementation, {
    onEnter: function(args) {
        this.url = ObjC.Object(args[2]).toString();
    },
    onLeave: function(retval) {
        if (this.url.indexOf("cydia") !== -1 || 
            this.url.indexOf("sileo") !== -1) {
            retval.replace(0x0);  // Cannot open
            console.log("[JB Bypass] URL denied: " + this.url);
        }
    }
});

// Hook fork() — should fail on non-jailbroken
Interceptor.attach(Module.findExportByName("libsystem_kernel.dylib", "fork"), {
    onLeave: function(retval) {
        retval.replace(-1);  // fork fails
        console.log("[JB Bypass] fork() denied");
    }
});

// Hook _dyld_get_image_name to hide injected libraries
Interceptor.attach(Module.findExportByName(null, "_dyld_get_image_name"), {
    onLeave: function(retval) {
        var name = retval.readUtf8String();
        if (name && (name.indexOf("Substrate") !== -1 || 
                     name.indexOf("frida") !== -1 ||
                     name.indexOf("cycript") !== -1)) {
            retval.replace(Memory.allocUtf8String("/usr/lib/libSystem.B.dylib"));
        }
    }
});
```

---

## 3. Advanced Detection vs. Bypass

```
CAT AND MOUSE GAME:

ADVANCED DETECTION:
  → Check at multiple points (not just startup)
  → Use inline checks (not separate function)
  → Verify in native code (.so/.dylib)
  → Integrity check detection code itself
  → Use syscalls directly (avoid libc hooks)
  → Combine multiple detection signals

ADVANCED BYPASS:
  → Hook at syscall level (lower than libc)
  → Patch binary (permanent modification)
  → Use kernel-level hooks (if available)
  → Combine multiple bypass tools

DETECTION MATURITY LEVELS:
┌───────┬──────────────────────┬──────────────────────┐
│ Level │ Detection            │ Bypass Difficulty    │
├───────┼──────────────────────┼──────────────────────┤
│ 1     │ Single file check    │ Trivial             │
│ 2     │ Multiple file checks │ Easy (objection)    │
│ 3     │ + URL scheme + fork  │ Medium (Frida)      │
│ 4     │ + Dylib check        │ Medium-Hard         │
│ 5     │ + Native code checks │ Hard (binary patch) │
│ 6     │ + Integrity verify   │ Very Hard           │
│ 7     │ + Server-side verify │ Extremely Hard      │
└───────┴──────────────────────┴──────────────────────┘
```

---

## 4. Reporting Jailbreak Detection

```
PENTESTING PERSPECTIVE:

IF DETECTION IS WEAK:
  → Report: "Jailbreak detection easily bypassed"
  → Show: Steps to bypass with objection/Frida
  → Impact: Attacker can run app on compromised device
  → Recommendation: Implement multiple detection layers

IF NO DETECTION:
  → Report: "No jailbreak detection implemented"
  → Impact: App runs normally on jailbroken devices
  → Risk: Easier runtime analysis and tampering
  → Recommendation: Add detection with server verification

NOTE: Jailbreak detection is defense-in-depth
  → It SLOWS attackers but doesn't STOP them
  → Should be combined with other protections
  → Root/jailbreak detection alone is NOT sufficient
  → Server-side security is the ultimate defense
```

---

## Summary Table

| Detection Method | Bypass Tool | Difficulty |
|-----------------|-------------|-----------|
| File existence | objection / Frida | Easy |
| URL schemes | objection / Frida | Easy |
| Sandbox write | Frida hook | Medium |
| Dylib detection | Frida / tweak | Medium |
| fork() check | Frida hook | Easy |
| Native code | Binary patching | Hard |
| Server-side | Cannot bypass client-side | Very Hard |

---

## Revision Questions

1. What are the common jailbreak detection methods used by iOS apps?
2. How does objection bypass jailbreak detection?
3. How do you write a comprehensive Frida script for jailbreak bypass?
4. What makes server-side jailbreak verification harder to bypass?
5. Why is jailbreak detection considered defense-in-depth rather than a security boundary?

---

*Previous: [04-traffic-interception.md](04-traffic-interception.md) | Next: [06-tools-objection-frida-hopper.md](06-tools-objection-frida-hopper.md)*

---

*[Back to README](../README.md)*

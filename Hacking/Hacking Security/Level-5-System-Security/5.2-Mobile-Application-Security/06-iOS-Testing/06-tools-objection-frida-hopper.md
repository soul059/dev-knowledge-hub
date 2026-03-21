# Unit 6: iOS Testing — Topic 6: Tools — Objection, Frida, and Hopper

## Overview

This topic provides a comprehensive reference for the three most important iOS security testing tools: **Frida** (dynamic instrumentation), **objection** (automated Frida-based testing), and **Hopper** (disassembler/decompiler). Together they cover the entire spectrum from runtime analysis to static binary reverse engineering.

---

## 1. Frida — Complete Reference

```
FRIDA ARCHITECTURE:

┌────────────────────────┐
│    Frida Client (PC)    │
│    frida-tools / Python │
└───────────┬─────────────┘
            │ USB / Network
┌───────────▼─────────────┐
│    frida-server          │
│    (runs on device)      │
├──────────────────────────┤
│    Target Process        │
│    ┌──────────────────┐  │
│    │  Injected Agent   │  │
│    │  (JavaScript V8)  │  │
│    │  Hooks, monitors  │  │
│    └──────────────────┘  │
└──────────────────────────┘

ESSENTIAL COMMANDS:
  frida-ps -U                      # List USB device processes
  frida-ps -Ua                     # Running apps
  frida-ps -Uai                    # All installed apps
  frida -U -f com.app -l script.js # Spawn with script
  frida -U -n "AppName"            # Attach to running
  frida-trace -U -f com.app -m "*[Login* *]"  # Trace methods
```

```javascript
// === FRIDA iOS COOKBOOK ===

// Enumerate all loaded ObjC classes
ObjC.enumerateLoadedClasses({
    onMatch: function(name) { console.log(name); },
    onComplete: function() { console.log("Done"); }
});

// Find classes matching pattern
ObjC.enumerateLoadedClassesSync()
    .filter(c => c.includes("Payment"))
    .forEach(c => console.log(c));

// List all methods of a class
var methods = ObjC.classes.ViewController.$ownMethods;
console.log(JSON.stringify(methods, null, 2));

// Hook instance method
Interceptor.attach(
    ObjC.classes.AuthManager['- isUserAuthenticated'].implementation, {
    onEnter: function(args) {
        console.log("[*] isUserAuthenticated called");
    },
    onLeave: function(retval) {
        console.log("[*] Original return: " + retval);
        retval.replace(0x1); // Force TRUE
    }
});

// Hook class method
Interceptor.attach(
    ObjC.classes.CryptoUtils['+ encryptString:withKey:'].implementation, {
    onEnter: function(args) {
        console.log("[*] Encrypting: " + ObjC.Object(args[2]));
        console.log("[*] Key: " + ObjC.Object(args[3]));
    }
});

// Create ObjC object
var NSString = ObjC.classes.NSString;
var myString = NSString.stringWithString_("injected");

// Call ObjC method
var app = ObjC.classes.UIApplication.sharedApplication();
var delegate = app.delegate();
console.log("Delegate: " + delegate.$className);

// Hook C function
Interceptor.attach(Module.findExportByName("libcommonCrypto.dylib", "CCCrypt"), {
    onEnter: function(args) {
        console.log("[CCCrypt] Op: " + args[0]);     // encrypt/decrypt
        console.log("[CCCrypt] Alg: " + args[1]);    // algorithm
        console.log("[CCCrypt] KeyLen: " + args[4]);  // key length
        // Dump key bytes
        console.log("[CCCrypt] Key: " + hexdump(args[3], {length: args[4].toInt32()}));
    }
});

// Read/write memory
var addr = Module.findExportByName(null, "some_function");
console.log(hexdump(addr, {offset: 0, length: 64}));

// Scan memory for pattern
Memory.scan(module.base, module.size, "48 89 5C 24", {
    onMatch: function(address, size) {
        console.log("Found at: " + address);
    },
    onComplete: function() {}
});
```

---

## 2. Objection — Automated Testing

```bash
# === OBJECTION COMMAND REFERENCE ===

# Connection
objection -g com.example.app explore
objection -g "App Name" explore
objection -g com.example.app explore --startup-command "ios sslpinning disable"

# === ENVIRONMENT ===
> env                              # Show app paths

# === KEYCHAIN ===
> ios keychain dump                # Dump all keychain items
> ios keychain dump --json         # JSON format
> ios keychain clear               # Clear keychain (careful!)

# === STORAGE ===
> ios nsuserdefaults get           # Dump UserDefaults
> ios plist cat <path>             # Read plist file
> ios cookies get                  # App cookies

# === BINARY INFO ===
> ios info binary                  # Binary details
> ios bundles list_frameworks      # Loaded frameworks

# === SECURITY BYPASSES ===
> ios sslpinning disable           # Disable cert pinning
> ios jailbreak disable            # Bypass jailbreak detect
> ios touchid bypass               # Bypass biometric

# === UI ===
> ios ui dump                      # View hierarchy
> ios ui alert "Test"              # Show alert
> ios ui screenshot                # Take screenshot

# === HOOKING ===
> ios hooking list classes                              # All classes
> ios hooking search classes Login                      # Search
> ios hooking list class_methods LoginViewController    # Methods
> ios hooking watch method "-[LoginVC login:]" \
    --dump-args --dump-return                           # Watch method
> ios hooking set return_value "-[LoginVC isAdmin]" true  # Force return

# === FILE SYSTEM ===
> ls /                             # List files
> !cat /path/to/file               # Read file
> file download /path/to/file ./   # Download file
> file upload ./local /device/path # Upload file

# === MEMORY ===
> memory dump all dump.bin         # Dump process memory
> memory search "password"         # Search in memory
> memory list modules              # Loaded modules
> memory list exports <module>     # Module exports
```

---

## 3. Hopper Disassembler

```
HOPPER FEATURES:

PURPOSE:
  → Disassemble Mach-O binaries
  → Decompile to pseudo-C code
  → Navigate cross-references
  → Patch binaries
  → Available for macOS and Linux

WORKFLOW:
  1. File → Open → Select Mach-O binary
  2. Select architecture (arm64 typically)
  3. Wait for analysis to complete
  4. Navigate code:
     → Search for strings (Shift+Cmd+F)
     → Find function by name
     → Follow cross-references
     → View decompiled code (Tab)

KEY FEATURES:
  Labels & Comments    → Annotate analysis
  Procedures           → Navigate functions
  Strings              → Find hardcoded values
  Cross-references     → Find callers/callees
  Pseudo-code          → Decompile to C
  Hex editor           → Patch bytes
  Segments             → View binary layout

FINDING INTERESTING FUNCTIONS:
  → Search: "jailbreak", "root", "detect"
  → Search: "pin", "certificate", "SSL"
  → Search: "encrypt", "decrypt", "AES"
  → Search: "password", "token", "auth"
  → Look at imported functions from Security.framework

PATCHING WITH HOPPER:
  1. Find function to patch
  2. Modify → Assemble Instruction
  3. Change conditional branch to unconditional
  4. Or NOP out entire check
  5. File → Produce New Executable
  6. Re-sign patched binary

EXAMPLE PATCH (Jailbreak detection):
  Before: CBZ X0, pass    (branch if zero = not jailbroken)
  After:  B pass           (always branch = always pass)
  
  Or: Change return value
  Before: MOV W0, #1      (return true = jailbroken)
  After:  MOV W0, #0      (return false = not jailbroken)
```

---

## 4. Tool Comparison

```
WHEN TO USE EACH TOOL:

┌──────────────┬────────────────────────────────┐
│ Task         │ Best Tool                      │
├──────────────┼────────────────────────────────┤
│ Quick recon  │ objection (automated commands) │
│ SSL bypass   │ objection → Frida if fails     │
│ JB bypass    │ objection → Frida if fails     │
│ Keychain     │ objection (ios keychain dump)   │
│ Custom hooks │ Frida (JavaScript scripting)    │
│ Crypto dump  │ Frida (hook CCCrypt)            │
│ Binary RE    │ Hopper (or Ghidra for free)     │
│ Patching     │ Hopper (produce new executable) │
│ Method trace │ frida-trace                     │
│ Automation   │ objection + Frida scripts       │
└──────────────┴────────────────────────────────┘
```

---

## Summary Table

| Tool | Type | Cost | Primary Use |
|------|------|------|-------------|
| Frida | Dynamic instrumentation | Free | Runtime hooking |
| objection | Frida automation | Free | Quick automated testing |
| Hopper | Disassembler | $99 | Binary analysis/patching |
| Ghidra | Disassembler | Free (NSA) | Alternative to Hopper |
| Cycript | Runtime exploration | Free | Interactive exploration |
| r2frida | radare2 + Frida | Free | Combined static+dynamic |

---

## Revision Questions

1. What are the key differences between Frida, objection, and Hopper?
2. How do you hook an Objective-C method with Frida?
3. What objection commands are essential for initial iOS assessment?
4. How do you patch a binary using Hopper?
5. When would you use Frida vs. objection for the same task?
6. How do you dump encryption keys using Frida on iOS?

---

*Previous: [05-jailbreak-detection-bypass.md](05-jailbreak-detection-bypass.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

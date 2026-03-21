# Unit 3: Android Testing — Topic 6: Tools (jadx, apktool, Frida)

## Overview

Three tools form the **core toolkit** for Android security testing: **jadx** for decompilation, **apktool** for disassembly/reassembly, and **Frida** for runtime instrumentation. Mastering these tools enables comprehensive static and dynamic analysis of any Android application.

---

## 1. jadx — Java Decompiler

```
jadx converts DEX bytecode back to readable Java source code.

INSTALLATION:
  → Download from github.com/skylot/jadx/releases
  → Requires Java 11+

USAGE:
  # GUI mode (recommended for code review)
  jadx-gui target.apk
  → Browse decompiled source in tree view
  → Search across all classes
  → Navigate imports/references
  → Export decompiled source

  # CLI mode
  jadx target.apk -d output_dir/
  jadx --deobf target.apk -d output_dir/  # Deobfuscation mode
  jadx --show-bad-code target.apk -d output_dir/  # Show even broken decompilation

KEY FEATURES:
  ├── Decompiles to readable Java
  ├── Handles obfuscated code (--deobf)
  ├── Search by string, class, method
  ├── Cross-reference navigation
  ├── Resource file viewing
  ├── AndroidManifest.xml decoding
  └── Export as Gradle project

WHAT TO LOOK FOR:
  → Strings containing "password", "secret", "key", "token"
  → URLs and API endpoints
  → Cryptographic implementations
  → Authentication/authorization logic
  → WebView configurations
  → Root detection code
  → Certificate pinning implementation
```

---

## 2. apktool — APK Disassembly/Reassembly

```
apktool decodes APK resources and converts DEX to smali (assembly).
CRITICAL for: modifying apps, bypassing protections, repackaging.

INSTALLATION:
  → Download from ibotpeaches.github.io/Apktool/

USAGE:
  # Decompile APK
  apktool d target.apk -o output_dir/
  # Produces:
  # ├── AndroidManifest.xml (decoded, readable)
  # ├── smali/               (Dalvik assembly code)
  # ├── res/                 (decoded resources)
  # └── assets/              (raw assets)

  # Rebuild APK after modifications
  apktool b output_dir/ -o modified.apk

  # Sign the rebuilt APK (required for installation)
  # Generate key:
  keytool -genkey -v -keystore test.jks -alias test -keyalg RSA -keysize 2048 -validity 10000
  # Sign:
  apksigner sign --ks test.jks --ks-key-alias test modified.apk
  # Or use jarsigner:
  jarsigner -verbose -sigalg SHA256withRSA -digestalg SHA-256 -keystore test.jks modified.apk test

  # Install modified APK
  adb install modified.apk

COMMON MODIFICATIONS:
  → Change android:debuggable to "true"
  → Add network security config (trust user certs)
  → Modify root detection smali code
  → Patch certificate pinning
  → Enable cleartext traffic
  → Modify application logic
```

```
SMALI BASICS:
  → Smali is Dalvik assembly language
  → Each .java file → .smali file
  → Modify method return values, add instructions

  # Example: Bypass isRooted() check
  # Find in smali: invoke-virtual {}, Lcom/example/Security;->isRooted()Z
  # Change method to always return false:
  
  .method public isRooted()Z
      .locals 1
      const/4 v0, 0x0    # 0 = false
      return v0
  .end method
```

---

## 3. Frida — Dynamic Instrumentation

```
Frida enables runtime code injection and hooking on running apps.
THE most powerful mobile testing tool.

INSTALLATION:
  pip install frida-tools

  # On device:
  # Download frida-server matching your Frida version and device arch
  adb push frida-server-16.x.x-android-arm64 /data/local/tmp/frida-server
  adb shell chmod 755 /data/local/tmp/frida-server
  adb shell /data/local/tmp/frida-server &

BASIC COMMANDS:
  frida-ps -U                          # List processes
  frida -U -f com.example.app          # Spawn and attach
  frida -U com.example.app             # Attach to running
  frida -U -f com.example.app -l script.js --no-pause  # Run script

FRIDA SCRIPT PATTERNS:

  # Hook Java method
  Java.perform(function() {
      var cls = Java.use("com.example.ClassName");
      cls.methodName.implementation = function(arg1) {
          console.log("Called with: " + arg1);
          var result = this.methodName(arg1);
          console.log("Returned: " + result);
          return result;
      };
  });

  # Hook overloaded method
  cls.methodName.overload('java.lang.String', 'int').implementation = function(s, i) {
      return this.methodName(s, i);
  };

  # Enumerate loaded classes
  Java.perform(function() {
      Java.enumerateLoadedClasses({
          onMatch: function(className) {
              if (className.includes("example")) {
                  console.log(className);
              }
          },
          onComplete: function() {}
      });
  });

  # Hook native function
  Interceptor.attach(Module.findExportByName("libnative.so", "checkRoot"), {
      onEnter: function(args) {
          console.log("checkRoot called");
      },
      onLeave: function(retval) {
          retval.replace(0);  // Force return 0 (false)
      }
  });
```

---

## 4. Tool Comparison

| Feature | jadx | apktool | Frida |
|---------|------|---------|-------|
| Analysis Type | Static | Static + Modify | Dynamic |
| Output | Java source | Smali + resources | Runtime hooks |
| Modification | Read-only | Full repackage | Runtime only |
| Best For | Code review | App modification | Runtime testing |
| Learning Curve | Easy | Medium | Medium-Hard |
| Persistence | N/A | Permanent (new APK) | Session only |

---

## Revision Questions

1. How do jadx, apktool, and Frida complement each other?
2. When would you use apktool over jadx for analysis?
3. Write a Frida script to intercept and log all SharedPreferences writes.
4. How do you modify smali code to bypass a boolean security check?
5. What is the workflow for repackaging an APK with apktool?

---

*Previous: [05-root-detection-bypass.md](05-root-detection-bypass.md) | Next: None (Final topic in this unit)*

---

*[Back to README](../README.md)*

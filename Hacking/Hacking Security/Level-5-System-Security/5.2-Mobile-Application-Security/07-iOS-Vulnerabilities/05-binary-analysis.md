# Unit 7: iOS Vulnerabilities — Topic 5: Binary Analysis

## Overview

**Binary analysis** of iOS apps involves examining the compiled Mach-O executable to understand app logic, find vulnerabilities, extract secrets, and assess binary protections. This includes both static analysis (disassembly, decompilation) and checking for security features like PIE, ARC, and stack canaries.

---

## 1. Mach-O Binary Format

```
MACH-O BINARY STRUCTURE:

┌────────────────────────────────────┐
│         MACH-O HEADER              │
│  Magic, CPU type, file type,       │
│  number of load commands           │
├────────────────────────────────────┤
│         LOAD COMMANDS              │
│  LC_SEGMENT_64 (__TEXT)            │
│  LC_SEGMENT_64 (__DATA)            │
│  LC_DYLD_INFO_ONLY                │
│  LC_SYMTAB                         │
│  LC_DYSYMTAB                       │
│  LC_LOAD_DYLIB                     │
│  LC_CODE_SIGNATURE                 │
│  LC_ENCRYPTION_INFO_64             │
├────────────────────────────────────┤
│         __TEXT SEGMENT              │
│  __text     (code)                 │
│  __stubs    (PLT stubs)            │
│  __stub_helper                     │
│  __cstring  (C strings)            │
│  __objc_methname  (method names)   │
│  __objc_classname (class names)    │
├────────────────────────────────────┤
│         __DATA SEGMENT              │
│  __objc_classlist                  │
│  __objc_protolist                  │
│  __objc_selrefs                    │
│  __cfstring                        │
│  __la_symbol_ptr                   │
├────────────────────────────────────┤
│         __LINKEDIT                  │
│  Symbol table, string table,       │
│  code signature                    │
└────────────────────────────────────┘

UNIVERSAL (FAT) BINARY:
  → Contains multiple architectures
  → arm64 (modern 64-bit)
  → armv7 (older 32-bit, deprecated)
  → x86_64 (simulator)
  
  lipo -info binary
  lipo binary -thin arm64 -output binary_arm64
```

---

## 2. Binary Security Checks

```bash
# === ESSENTIAL BINARY SECURITY CHECKS ===

# Check binary type and architecture
file MyApp
# Expected: Mach-O 64-bit arm64 executable

# Check for PIE (Position Independent Executable)
otool -hv MyApp | grep PIE
# MISSING PIE = ASLR not effective → FINDING!

# Check for ARC (Automatic Reference Counting)
otool -Iv MyApp | grep -c "objc_release"
# 0 results = no ARC → potential memory corruption

# Check for stack canaries
otool -Iv MyApp | grep "stack_chk"
# Missing = no stack smashing protection → FINDING!

# Check encryption (App Store binaries)
otool -l MyApp | grep -A4 "LC_ENCRYPTION_INFO"
# cryptid 1 = encrypted (need to decrypt first)
# cryptid 0 = decrypted (or manually decrypted)

# List linked libraries
otool -L MyApp
# Check for: insecure libraries, known vulnerable versions

# Check for dangerous functions
otool -Iv MyApp | grep -E "strlen|strcpy|strcat|sprintf|gets|scanf"
# These C functions are buffer overflow risks

# Comprehensive check with rabin2
rabin2 -I MyApp
# Shows: arch, bits, canary, crypto, nx, pic, stripped

# Check Objective-C metadata
otool -oV MyApp | head -100
# Class and method information
```

---

## 3. Disassembly and Decompilation

```
TOOL COMPARISON:

┌──────────┬────────┬──────────────────────────┐
│ Tool     │ Cost   │ Strengths                │
├──────────┼────────┼──────────────────────────┤
│ Ghidra   │ Free   │ Best free decompiler     │
│ IDA Pro  │ $$$    │ Industry standard        │
│ Hopper   │ $99    │ macOS-native, fast       │
│ radare2  │ Free   │ CLI, scriptable          │
│ Cutter   │ Free   │ r2 GUI, Ghidra decompiler│
│ Binary   │ $$$    │ Cloud-based, fast        │
│  Ninja   │        │                          │
└──────────┴────────┴──────────────────────────┘

GHIDRA WORKFLOW:
  1. Import Mach-O binary
  2. Select arm64 architecture
  3. Auto-analyze (wait for completion)
  4. Navigate to functions of interest:
     → Search strings for "password", "encrypt", "token"
     → Find ObjC method implementations
     → Follow cross-references
     → View decompiled C code
  
  KEY AREAS TO ANALYZE:
  → Authentication functions
  → Encryption/decryption routines
  → Key generation and storage
  → Jailbreak detection logic
  → Certificate pinning implementation
  → License/premium verification
```

```bash
# === CLASS-DUMP (ObjC headers) ===
class-dump MyApp > headers.h
# Reveals:
# → All class names
# → All method signatures
# → Property declarations
# → Protocol implementations

# Search for interesting methods
grep -i "password\|credential\|auth\|encrypt\|decrypt\|admin\|premium" headers.h

# === RADARE2 ANALYSIS ===
r2 -A MyApp
> afl                    # List functions
> afl~password           # Find password-related functions
> s sym.func_name        # Seek to function
> pdf                    # Print disassembly
> VV                     # Visual graph mode
> iz                     # List strings
> iz~http                # Find URLs
```

---

## 4. String and Secret Extraction

```bash
# Extract all strings
strings MyApp > strings.txt

# Search for secrets
grep -iE "password|secret|key|token|api.key|private" strings.txt
grep -E "https?://" strings.txt           # URLs and endpoints
grep -E "[A-Za-z0-9+/]{40,}" strings.txt  # Base64 encoded data
grep -E "AKIA[0-9A-Z]{16}" strings.txt    # AWS access keys
grep -E "sk_live_" strings.txt            # Stripe keys
grep -E "AIza[0-9A-Za-z_-]{35}" strings.txt  # Google API keys

# Extract from specific sections
otool -s __TEXT __cstring MyApp         # C strings
otool -s __DATA __cfstring MyApp        # CF strings
otool -s __TEXT __objc_methname MyApp   # Method names

# Using rabin2
rabin2 -z MyApp        # Strings from data section
rabin2 -zz MyApp       # All strings (including code)
rabin2 -zqq MyApp | grep -i "api\|key\|secret\|password"
```

---

## 5. Binary Protection Assessment

```
SECURITY CHECKLIST:

┌─────────────────────┬──────────┬───────────────────────┐
│ Protection          │ Required │ Check Command          │
├─────────────────────┼──────────┼───────────────────────┤
│ PIE (ASLR)          │ Yes      │ otool -hv (MH_PIE)    │
│ ARC                 │ Yes      │ otool -Iv (objc_rel.)  │
│ Stack Canaries      │ Yes      │ otool -Iv (stack_chk)  │
│ Code Signing        │ Yes      │ codesign -dvvv         │
│ Encryption (Store)  │ Auto     │ otool -l (cryptid)     │
│ Stripped Symbols    │ Recom.   │ otool -Iv (count)      │
│ No Dangerous Funcs  │ Recom.   │ otool -Iv (strcpy etc) │
│ Bitcode             │ Optional │ otool -l (LLVM)        │
│ Rpath               │ Check    │ otool -l (LC_RPATH)    │
└─────────────────────┴──────────┴───────────────────────┘

REPORTING:
  CRITICAL: No PIE → ASLR ineffective
  HIGH: No stack canaries → Buffer overflow exploitable
  MEDIUM: No ARC → Memory corruption risk
  LOW: Symbols not stripped → Easier reverse engineering
  INFO: Dangerous C functions used → Manual review needed
```

---

## Summary Table

| Analysis | Tool | Purpose |
|----------|------|---------|
| Format inspection | otool, rabin2 | Binary metadata |
| Security checks | otool, rabin2 | PIE, ARC, canaries |
| Disassembly | Ghidra, Hopper, r2 | Code analysis |
| Class headers | class-dump | ObjC interface |
| String extraction | strings, rabin2 | Secrets, URLs |
| Decryption | frida-ios-dump | App Store binaries |

---

## Revision Questions

1. What is the Mach-O binary format and its key segments?
2. How do you check for PIE, ARC, and stack canaries?
3. What tools are used for iOS binary disassembly?
4. How do you extract secrets from compiled binaries?
5. What is the severity of missing binary protections?

---

*Previous: [04-runtime-manipulation.md](04-runtime-manipulation.md) | Next: [06-certificate-pinning-bypass.md](06-certificate-pinning-bypass.md)*

---

*[Back to README](../README.md)*
